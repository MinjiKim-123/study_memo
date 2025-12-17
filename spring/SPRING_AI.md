# Spring AI란?

Spring에서 AI 엔지니어링을 하기 위한 프레임워크. <br/>
Spring에서 AI를 간단하게 활용할 수 있도록 도와주며, spring의 설계 원칙을 AI도메인에 적용.

## ChatModel
AI 기반의 채팅 완성 기능을 애플리케이션에 통합할 수 있도록 지원.<br/>
LLM API 호출을 추상화한 인터페이스를 제공하여 단순하고 재활용성이 높은 코드 작성이 가능. (AI모델 변경 용이)
<br/>
```
    public static <T> T call(ChatModel chatModel, String userMessage, String systemMessage, Class<T> returnType) {
        ChatClient chatClient = ChatClient.create(chatModel);
    	List<Message> messages = List.of(new SystemMessage(systemMessage), new UserMessage(userMessage));
        Prompt chatPrompt = new Prompt(messages);
    	return chatClient
    			.prompt(chatPrompt)
    			.call()
    			.entity(returnType);
    }
```
<sup>*호출 유틸리티 코드 작성 예시(chatoptions이나 advisor같은 추가 설정은 제외한 기본 설정만 세팅하였음.)</sup>

+ 동작방식
<img src="https://docs.spring.io/spring-ai/reference/_images/chat-options-flow.jpg" width="500px"/>
<sup>출처: 공식문서</sup><br/>
1. 채팅 옵션 및 메세지 프롬포트 구성. (Chatoption은 runtime 옵션과 시작 옵션을 병합하여 호출됨.) <br/>
2. 입력 값 처리 (요청 객체 생성) <br/>
3. AI 호출 (API 호출로 보이는데 다른 방식도 있나..?) <br/>
4. 응답 값 처리 <br/>
- call().content() -> string 반환 <br/>
- call().entity(T) -> T type 반환 <br/>

```
	@Override
		@Nullable
		public <T> T entity(Class<T> type) {
			Assert.notNull(type, "type cannot be null");
			var outputConverter = new BeanOutputConverter<>(type);
			return doSingleWithBeanOutputConverter(outputConverter);
		}
```

entity 호출시 내부에서 outputConverter를 미리 설정함.

+ OutputConverter
<br/>LLM출력을 구조화된 형식으로 변환하는데 도움을 주는 변환기. 현재는 아래 5가지를 제공해줌. (spring-ai ver.1.0.0-M6 기준)
<img src="https://docs.spring.io/spring-ai/reference/_images/structured-output-hierarchy4.jpg" width="500px"/>
<sup>출처: 공식문서</sup><br/>
1. AbstractConversionServiceOutputConverter <br>
LLM 출력 결과를 GenericConversionService를 사용해서 원하는 결과로 convert 할 수 있도록 제공하는 추상클래스.
2. AbstractMessageOutputConverter <br>
LLM 출력 결과를 MessageConvert를 사용해서 원하는 결과로 convert 할 수 있도록 제공하는 추상클래스.
3. BeanOutputConverter <br>
4. MapOutputConverter<br>
5. ListOutputConverter<br>
1번 AbstractConversionServiceOutputConverter 클래스를 확장한 Converter.<br>
<br>


Converter 구현체를 보면 아래와 같이 프롬포트를 설정해서 원하는 형식의 응답을 받고있으며,<br>
각 Converter 형식에 따라 구현된 getFormat의 내용이 다름
 ```
	/**
	 * Provides the expected format of the response, instructing that it should adhere to
	 * the generated JSON schema.
	 * @return The instruction format string.
	 */
	@Override
	public String getFormat() {
		String template = """
				Your response should be in JSON format.
				...
				Here is the JSON Schema instance your output must adhere to:
				```%s```
				""";
		return String.format(template, this.jsonSchema);
	}
```
<sup>*BeanOutputConverter class의 getFormat 메소드</sup>

  
