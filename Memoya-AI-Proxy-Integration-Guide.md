# 📚 Memoya AI 프록시 서버 연동 가이드

외부 개발자를 위한 완전한 연동 매뉴얼

---

## 🎯 개요

**Memoya AI 프록시 서버**는 Google Gemini API를 안전하게 사용할 수 있는 프록시 서비스입니다. API 키를 클라이언트에 노출하지 않고도 AI 기능을 앱에 통합할 수 있습니다.

### ✅ 지원 기능
- Google Gemini AI 모든 모델 지원
- Function Calling (도구 사용) 완벽 지원
- 다국어 대화 지원
- 실시간 대화 처리
- 안전한 API 키 관리

### 🌐 서버 정보
- **Base URL**: `https://memoya-proxy-production.up.railway.app`
- **Status**: 24/7 운영 중
- **Rate Limit**: Gemini API 유료 플랜 기준 (분당 1,000 requests)
- **지원 모델**: 모든 Gemini 모델 (앱에서 선택)

---

## 🚀 빠른 시작

### 1. 기본 AI 채팅 연동

```javascript
// 기본 AI 채팅 요청
const sendMessage = async (userMessage) => {
  const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      contents: [
        {
          parts: [{ text: userMessage }],
          role: 'user'
        }
      ]
    })
  });
  
  const data = await response.json();
  return data.candidates[0].content.parts[0].text;
};

// 사용 예제
const aiResponse = await sendMessage("안녕하세요! 오늘 날씨가 어때요?");
console.log(aiResponse); // AI의 응답
```

### 2. 모델 선택하기

```javascript
// 특정 모델 사용 (URL에 모델 지정)
const sendMessageWithModel = async (userMessage, model = 'gemini-1.5-flash-8b') => {
  // 클라이언트에서 모델 결정
  const API_URL = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent`;
  
  const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      contents: [
        {
          parts: [{ text: userMessage }],
          role: 'user'
        }
      ]
    })
  });
  
  return response.json();
};
```

**사용 가능한 모델:**
- `gemini-1.5-flash-8b` (기본, 빠름, 저비용)
- `gemini-1.5-pro` (고성능, 복잡한 작업)
- `gemini-2.0-flash-exp` (실험적, 최신 기능)

---

## 💬 고급 기능

### 3. 대화 이력 유지

```javascript
class AIChat {
  constructor() {
    this.chatHistory = [];
  }
  
  async sendMessage(userMessage, systemPrompt = '') {
    // 시스템 프롬프트 추가 (선택사항)
    const messages = [];
    if (systemPrompt) {
      messages.push({
        parts: [{ text: systemPrompt }],
        role: 'user'
      });
    }
    
    // 대화 이력 추가
    messages.push(...this.chatHistory);
    
    // 새 사용자 메시지 추가
    messages.push({
      parts: [{ text: userMessage }],
      role: 'user'
    });
    
    const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        contents: messages
      })
    });
    
    const data = await response.json();
    const aiResponse = data.candidates[0].content.parts[0].text;
    
    // 대화 이력 업데이트
    this.chatHistory.push(
      { parts: [{ text: userMessage }], role: 'user' },
      { parts: [{ text: aiResponse }], role: 'model' }
    );
    
    return aiResponse;
  }
  
  clearHistory() {
    this.chatHistory = [];
  }
}

// 사용 예제
const chat = new AIChat();
await chat.sendMessage("안녕하세요!", "당신은 도움이 되는 AI 어시스턴트입니다.");
await chat.sendMessage("제 이름은 홍길동입니다.");
await chat.sendMessage("제 이름이 뭐였죠?"); // "홍길동입니다" 라고 기억함
```

### 4. Function Calling (도구 사용)

```javascript
// AI가 사용할 수 있는 도구 정의
const tools = [
  {
    functionDeclarations: [
      {
        name: "get_weather",
        description: "현재 날씨 정보를 가져옵니다",
        parameters: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "날씨를 확인할 도시명"
            }
          },
          required: ["location"]
        }
      },
      {
        name: "calculate",
        description: "수학 계산을 수행합니다",
        parameters: {
          type: "object",
          properties: {
            expression: {
              type: "string",
              description: "계산할 수식 (예: 2+3*4)"
            }
          },
          required: ["expression"]
        }
      }
    ]
  }
];

const sendMessageWithTools = async (userMessage) => {
  let messages = [
    {
      parts: [{ text: userMessage }],
      role: 'user'
    }
  ];
  
  while (true) {
    const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        contents: messages,
        tools: tools,
        toolConfig: {
          functionCallingConfig: {
            mode: "ANY" // AI가 필요시 도구 사용
          }
        }
      })
    });
    
    const data = await response.json();
    const candidate = data.candidates[0];
    const parts = candidate.content.parts;
    
    // Function Call 확인
    const functionCall = parts.find(part => part.functionCall);
    
    if (functionCall) {
      // 도구 실행
      const result = await executeFunction(functionCall.functionCall);
      
      // 결과를 AI에게 전달
      messages.push(
        { parts: [functionCall], role: 'model' },
        { 
          parts: [{
            functionResponse: {
              name: functionCall.functionCall.name,
              response: result
            }
          }], 
          role: 'function' 
        }
      );
    } else {
      // 최종 응답
      return parts[0].text;
    }
  }
};

// 도구 실행 함수
const executeFunction = async (functionCall) => {
  switch (functionCall.name) {
    case 'get_weather':
      return {
        location: functionCall.args.location,
        temperature: '23°C',
        condition: '맑음'
      };
      
    case 'calculate':
      try {
        const result = eval(functionCall.args.expression);
        return { result: result };
      } catch (error) {
        return { error: '계산 오류' };
      }
      
    default:
      return { error: '알 수 없는 함수' };
  }
};

// 사용 예제
const response = await sendMessageWithTools("서울 날씨 어때? 그리고 2+3*4 계산해줘");
// AI가 자동으로 get_weather와 calculate 함수를 호출하여 답변
```

---

## 🛠️ 플랫폼별 구현

### React Native

```typescript
import { useState, useCallback } from 'react';

interface AIMessage {
  id: string;
  text: string;
  isUser: boolean;
  timestamp: Date;
}

export const useAI = () => {
  const [messages, setMessages] = useState<AIMessage[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  const sendMessage = useCallback(async (userMessage: string) => {
    setIsLoading(true);
    
    // 사용자 메시지 추가
    const userMsg: AIMessage = {
      id: Date.now().toString(),
      text: userMessage,
      isUser: true,
      timestamp: new Date()
    };
    setMessages(prev => [...prev, userMsg]);

    try {
      const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          contents: [
            {
              parts: [{ text: userMessage }],
              role: 'user'
            }
          ]
        })
      });

      const data = await response.json();
      const aiResponse = data.candidates[0].content.parts[0].text;

      // AI 응답 추가
      const aiMsg: AIMessage = {
        id: (Date.now() + 1).toString(),
        text: aiResponse,
        isUser: false,
        timestamp: new Date()
      };
      setMessages(prev => [...prev, aiMsg]);

    } catch (error) {
      console.error('AI 오류:', error);
      // 오류 처리
    } finally {
      setIsLoading(false);
    }
  }, []);

  return {
    messages,
    sendMessage,
    isLoading
  };
};
```

### Flutter

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class AIService {
  static const String _baseUrl = 'https://memoya-proxy-production.up.railway.app';
  
  static Future<String> sendMessage(String userMessage) async {
    final response = await http.post(
      Uri.parse('$_baseUrl/api/gemini'),
      headers: {
        'Content-Type': 'application/json',
      },
      body: jsonEncode({
        'contents': [
          {
            'parts': [{'text': userMessage}],
            'role': 'user'
          }
        ]
      }),
    );
    
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body);
      return data['candidates'][0]['content']['parts'][0]['text'];
    } else {
      throw Exception('AI 서비스 오류');
    }
  }
}

// 사용 예제
class ChatScreen extends StatefulWidget {
  @override
  _ChatScreenState createState() => _ChatScreenState();
}

class _ChatScreenState extends State<ChatScreen> {
  final TextEditingController _controller = TextEditingController();
  List<String> messages = [];
  bool isLoading = false;

  Future<void> _sendMessage() async {
    if (_controller.text.isEmpty) return;
    
    final userMessage = _controller.text;
    _controller.clear();
    
    setState(() {
      messages.add('사용자: $userMessage');
      isLoading = true;
    });
    
    try {
      final aiResponse = await AIService.sendMessage(userMessage);
      setState(() {
        messages.add('AI: $aiResponse');
      });
    } catch (error) {
      setState(() {
        messages.add('오류: $error');
      });
    } finally {
      setState(() {
        isLoading = false;
      });
    }
  }
}
```

### Web (JavaScript/TypeScript)

```html
<!DOCTYPE html>
<html>
<head>
    <title>AI Chat</title>
</head>
<body>
    <div id="chat-container"></div>
    <input type="text" id="message-input" placeholder="메시지를 입력하세요">
    <button onclick="sendMessage()">전송</button>

    <script>
        let chatHistory = [];
        
        async function sendMessage() {
            const input = document.getElementById('message-input');
            const userMessage = input.value.trim();
            if (!userMessage) return;
            
            // UI 업데이트
            addMessageToChat('사용자', userMessage);
            input.value = '';
            
            try {
                const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        contents: [
                            {
                                parts: [{ text: userMessage }],
                                role: 'user'
                            }
                        ]
                    })
                });
                
                const data = await response.json();
                const aiResponse = data.candidates[0].content.parts[0].text;
                
                addMessageToChat('AI', aiResponse);
                
            } catch (error) {
                addMessageToChat('시스템', '오류가 발생했습니다: ' + error.message);
            }
        }
        
        function addMessageToChat(sender, message) {
            const container = document.getElementById('chat-container');
            const messageDiv = document.createElement('div');
            messageDiv.innerHTML = `<strong>${sender}:</strong> ${message}`;
            container.appendChild(messageDiv);
            container.scrollTop = container.scrollHeight;
        }
        
        // Enter 키 처리
        document.getElementById('message-input').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                sendMessage();
            }
        });
    </script>
</body>
</html>
```

---

## 🔧 에러 처리 및 디버깅

### 일반적인 에러와 해결책

```javascript
const sendMessageWithErrorHandling = async (userMessage) => {
  try {
    const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        contents: [
          {
            parts: [{ text: userMessage }],
            role: 'user'
          }
        ]
      })
    });
    
    // HTTP 에러 체크
    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
    }
    
    const data = await response.json();
    
    // Gemini API 에러 체크
    if (data.error) {
      throw new Error(`Gemini API Error: ${data.error.message}`);
    }
    
    // 응답 구조 확인
    if (!data.candidates || !data.candidates[0] || !data.candidates[0].content) {
      throw new Error('잘못된 응답 형식');
    }
    
    return data.candidates[0].content.parts[0].text;
    
  } catch (error) {
    console.error('AI 서비스 오류:', error);
    
    // 에러 유형별 처리
    if (error.message.includes('HTTP Error: 429')) {
      return '너무 많은 요청입니다. 잠시 후 다시 시도해주세요.';
    } else if (error.message.includes('HTTP Error: 500')) {
      return '서버 오류가 발생했습니다. 나중에 다시 시도해주세요.';
    } else if (error.message.includes('Network')) {
      return '네트워크 연결을 확인해주세요.';
    } else {
      return '알 수 없는 오류가 발생했습니다.';
    }
  }
};
```

### 서버 상태 확인

```javascript
// 프록시 서버 상태 확인
const checkServerStatus = async () => {
  try {
    const response = await fetch('https://memoya-proxy-production.up.railway.app/');
    const status = await response.json();
    
    console.log('서버 상태:', status);
    /*
    출력 예시:
    {
      "server": "Memoya Proxy Server",
      "version": "1.0.0",
      "status": "정상 작동 중",
      "timestamp": "2025-08-31T14:44:23.249Z"
    }
    */
    
    return status.status === '정상 작동 중';
    
  } catch (error) {
    console.error('서버 상태 확인 실패:', error);
    return false;
  }
};

// 사용 예제
if (await checkServerStatus()) {
  console.log('서버가 정상 작동 중입니다.');
} else {
  console.log('서버에 문제가 있습니다.');
}
```

---

## 🎛️ 고급 설정

### Rate Limiting 고려사항

```javascript
// 요청 간격 조절 (Rate Limiting 방지)
class AIServiceWithRateLimit {
  constructor(minInterval = 1000) { // 최소 1초 간격
    this.minInterval = minInterval;
    this.lastRequestTime = 0;
  }
  
  async sendMessage(userMessage) {
    // 최소 간격 보장
    const now = Date.now();
    const timeSinceLastRequest = now - this.lastRequestTime;
    
    if (timeSinceLastRequest < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastRequest)
      );
    }
    
    this.lastRequestTime = Date.now();
    
    // 실제 요청
    return await this.makeRequest(userMessage);
  }
  
  async makeRequest(userMessage) {
    // 기본 요청 로직...
  }
}
```

### 캐싱 구현

```javascript
// 응답 캐싱으로 성능 향상
class AIServiceWithCache {
  constructor() {
    this.cache = new Map();
    this.maxCacheSize = 100;
  }
  
  getCacheKey(userMessage) {
    return btoa(userMessage.toLowerCase().trim()); // Base64 인코딩
  }
  
  async sendMessage(userMessage) {
    const cacheKey = this.getCacheKey(userMessage);
    
    // 캐시에서 확인
    if (this.cache.has(cacheKey)) {
      console.log('캐시에서 응답 반환');
      return this.cache.get(cacheKey);
    }
    
    // API 호출
    const response = await this.makeAPICall(userMessage);
    
    // 캐시 크기 관리
    if (this.cache.size >= this.maxCacheSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    // 캐시 저장
    this.cache.set(cacheKey, response);
    
    return response;
  }
}
```

---

## 📊 모니터링 및 로깅

### 사용량 추적

```javascript
// 사용량 통계 수집
class AIServiceWithAnalytics {
  constructor() {
    this.stats = {
      totalRequests: 0,
      successfulRequests: 0,
      failedRequests: 0,
      averageResponseTime: 0,
      lastUsed: null
    };
  }
  
  async sendMessage(userMessage) {
    const startTime = Date.now();
    this.stats.totalRequests++;
    this.stats.lastUsed = new Date();
    
    try {
      const response = await this.makeAPICall(userMessage);
      this.stats.successfulRequests++;
      
      // 응답 시간 계산
      const responseTime = Date.now() - startTime;
      this.updateAverageResponseTime(responseTime);
      
      return response;
      
    } catch (error) {
      this.stats.failedRequests++;
      throw error;
    }
  }
  
  updateAverageResponseTime(newTime) {
    const { successfulRequests, averageResponseTime } = this.stats;
    this.stats.averageResponseTime = 
      (averageResponseTime * (successfulRequests - 1) + newTime) / successfulRequests;
  }
  
  getStats() {
    return {
      ...this.stats,
      successRate: (this.stats.successfulRequests / this.stats.totalRequests * 100).toFixed(2) + '%'
    };
  }
}

// 사용 예제
const aiService = new AIServiceWithAnalytics();
await aiService.sendMessage("안녕하세요!");

console.log(aiService.getStats());
/*
{
  totalRequests: 1,
  successfulRequests: 1,
  failedRequests: 0,
  averageResponseTime: 1234,
  lastUsed: 2025-08-31T15:30:00.000Z,
  successRate: "100.00%"
}
*/
```

---

## 🚀 최적화 팁

### 1. 메시지 최적화
```javascript
// 긴 메시지는 요약해서 전송
const optimizeMessage = (message) => {
  if (message.length > 1000) {
    return message.substring(0, 997) + "...";
  }
  return message;
};
```

### 2. 배치 처리
```javascript
// 여러 요청을 배치로 처리
const processBatch = async (messages) => {
  const batchPromises = messages.map((msg, index) => 
    new Promise(resolve => 
      setTimeout(() => sendMessage(msg).then(resolve), index * 1000)
    )
  );
  
  return Promise.all(batchPromises);
};
```

### 3. 스트리밍 응답 (미래 기능)
```javascript
// 실시간 응답 스트리밍 (향후 지원 예정)
const streamMessage = async (userMessage, onChunk) => {
  // 현재는 일반 응답만 지원
  // 향후 Server-Sent Events나 WebSocket으로 스트리밍 지원 계획
  const response = await sendMessage(userMessage);
  onChunk(response);
};
```

---

## 📝 추가 예제 코드

### React Hook 형태 완전 구현

```typescript
// hooks/useMemoyaAI.ts
import { useState, useCallback, useRef } from 'react';

interface AIMessage {
  id: string;
  content: string;
  role: 'user' | 'assistant';
  timestamp: Date;
}

interface AIConfig {
  model?: string;
  systemPrompt?: string;
  maxHistory?: number;
}

export const useMemoyaAI = (config: AIConfig = {}) => {
  const [messages, setMessages] = useState<AIMessage[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const abortControllerRef = useRef<AbortController | null>(null);

  const {
    model = 'gemini-1.5-flash-8b',
    systemPrompt,
    maxHistory = 20
  } = config;

  const sendMessage = useCallback(async (content: string): Promise<string | null> => {
    if (!content.trim()) return null;

    setIsLoading(true);
    setError(null);

    // 이전 요청 취소
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();

    // 사용자 메시지 추가
    const userMessage: AIMessage = {
      id: Date.now().toString(),
      content: content.trim(),
      role: 'user',
      timestamp: new Date()
    };

    setMessages(prev => [...prev, userMessage]);

    try {
      // 대화 이력 준비
      const recentMessages = messages.slice(-maxHistory);
      const contents = [];

      // 시스템 프롬프트 추가
      if (systemPrompt) {
        contents.push({
          parts: [{ text: systemPrompt }],
          role: 'user'
        });
      }

      // 이전 대화 추가
      recentMessages.forEach(msg => {
        contents.push({
          parts: [{ text: msg.content }],
          role: msg.role === 'user' ? 'user' : 'model'
        });
      });

      // 현재 사용자 메시지 추가
      contents.push({
        parts: [{ text: content }],
        role: 'user'
      });

      const response = await fetch('https://memoya-proxy-production.up.railway.app/api/gemini', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ contents }),
        signal: abortControllerRef.current.signal
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }

      const data = await response.json();

      if (data.error) {
        throw new Error(data.error.message);
      }

      const assistantContent = data.candidates?.[0]?.content?.parts?.[0]?.text;

      if (!assistantContent) {
        throw new Error('응답 형식이 올바르지 않습니다');
      }

      // AI 응답 추가
      const assistantMessage: AIMessage = {
        id: (Date.now() + 1).toString(),
        content: assistantContent,
        role: 'assistant',
        timestamp: new Date()
      };

      setMessages(prev => [...prev, assistantMessage]);
      return assistantContent;

    } catch (err: any) {
      if (err.name === 'AbortError') {
        return null;
      }

      const errorMessage = err.message || '알 수 없는 오류가 발생했습니다';
      setError(errorMessage);
      
      // 에러 메시지를 채팅에 표시
      const errorMessageObj: AIMessage = {
        id: (Date.now() + 2).toString(),
        content: `오류: ${errorMessage}`,
        role: 'assistant',
        timestamp: new Date()
      };

      setMessages(prev => [...prev, errorMessageObj]);
      return null;

    } finally {
      setIsLoading(false);
      abortControllerRef.current = null;
    }
  }, [messages, maxHistory, systemPrompt]);

  const clearMessages = useCallback(() => {
    setMessages([]);
    setError(null);
  }, []);

  const cancelRequest = useCallback(() => {
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
  }, []);

  return {
    messages,
    sendMessage,
    clearMessages,
    cancelRequest,
    isLoading,
    error
  };
};
```

---

## ⚠️ 중요 사항 및 제한사항

### 보안 고려사항
1. **API 키 노출 금지**: 클라이언트 코드에 절대 API 키를 하드코딩하지 마세요
2. **HTTPS 필수**: 모든 요청은 HTTPS를 통해서만 전송됩니다
3. **입력 검증**: 사용자 입력은 항상 검증하고 sanitize하세요

### 사용 제한
1. **Rate Limiting**: 분당 1,000 requests (Gemini API 유료 플랜 기준)
2. **메시지 길이**: 단일 요청당 최대 30,000자
3. **대화 이력**: 권장 최대 20-30 메시지

### 비용 정보
- 프록시 서버 사용료: 무료
- Gemini API 비용은 실제 사용량에 따라 Google에서 과금
- 현재 개발자(memoya)가 API 비용 부담

### 지원 및 문의
- **이슈 리포트**: GitHub Issues (링크 제공 시)
- **기술 지원**: 개발자 연락처 (필요시 제공)
- **업데이트**: 서버 업데이트나 변경사항은 사전 공지

---

## 🔚 마무리

이 가이드를 통해 Memoya AI 프록시 서버를 성공적으로 연동하실 수 있습니다. 추가 질문이나 지원이 필요하시면 언제든 연락주세요!

**Happy Coding! 🚀**