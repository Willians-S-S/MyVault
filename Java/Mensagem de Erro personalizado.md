Para criar mensagens de erros personalizados devemos criar uma classe que manipulará as exceções, que terá como anotação _@ControllerAdvice_ essa anotação faz com que a classe capture todas a exceções a partir do métodos que colocaremos a anotação _@ExceptionHandler_ com a classe de erro que queremos capturar. 

```java
package com.willians.ListifyShop.exception;  
  
import jakarta.servlet.http.HttpServletRequest;  
import org.springframework.http.HttpStatus;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.ControllerAdvice;  
import org.springframework.web.bind.annotation.ExceptionHandler;  
import org.springframework.web.server.ResponseStatusException;  
  
import java.time.Instant;  
  
@ControllerAdvice  
public class CustomControllerAdvice {  
  
    @ExceptionHandler(ResponseStatusException.class)  
    public ResponseEntity<MessageError> error(ResponseStatusException e, HttpServletRequest request){  
        int ponto = e.getMessage().indexOf("\"") - 1;  
        int pontoFim = e.getMessage().length() - 1;  
        String error = e.getMessage().substring(0, ponto);  
        String message = e.getMessage().substring(ponto + 2, pontoFim);  
        return ResponseEntity.status(e.getStatusCode()).body(  
                new MessageError(  
                        Instant.now(),  
                        e.getStatusCode().value(),  
                        error,  
                        message,  
                        request.getRequestURI()  
                        ));  
    }  
  
    @ExceptionHandler(NotFoundException.class)  
    public ResponseEntity<MessageError> notFoundExceptionResponseEntity(NotFoundException e, HttpServletRequest request){  
        int cod = HttpStatus.NOT_FOUND.value();  
        return ResponseEntity.status(cod).body(  
                new MessageError(  
                    Instant.now(),  
                    cod,  
                    "Recurso não encontrado",  
                    e.getMessage(),  
                    request.getRequestURI()  
                    ));  
    }  
}
```

No código é possível ver que toda vez que for lançado os erros das classes _ResponseStatusException_ e _NotFoundException_, eles são capturados e são formatados

 _MessageError_ é um record no qual é composta pelos campos do retorno:
```java
package com.willians.ListifyShop.exception;  
  
import java.time.Instant;  
  
public record MessageError (Instant timestamp, Integer status, String error, String message, String path){}
```

O erro personalizado _NotFoundException_ pode ser analisado em [[Exceptions personalizada]]

## Mensagem de erro personalizado para as anotações
A mensagem de erro gerada pela anotação [[Anotação para validação em Java]] é longa e traz detalhes da aplicação como demostrado abaixo:

```json
{
  "timestamp": "2025-07-13T19:54:35.530+00:00",
  "status": 400,
  "error": "Bad Request",
  "trace": "org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public com.willians.ListifyShop.dto.UserResponseDto com.willians.ListifyShop.controller.UserController.addUser(com.willians.ListifyShop.dto.UserRequestDto): [Field error in object 'userRequestDto' on field 'cpf': rejected value [1111111111]; codes [CpfValid.userRequestDto.cpf,CpfValid.cpf,CpfValid.java.lang.String,CpfValid]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [userRequestDto.cpf,cpf]; arguments []; default message [cpf]]; default message [Cpf inválido.]] \n\tat org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor.resolveArgument(RequestResponseBodyMethodProcessor.java:159)\n\tat org.springframework.web.method.support.HandlerMethodArgumentResolverComposite.resolveArgument(HandlerMethodArgumentResolverComposite.java:122)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.getMethodArgumentValues(InvocableHandlerMethod.java:227)\n\tat org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:181)\n\tat org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:118)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:986)\n\tat org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:891)\n\tat org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)\n\tat org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1089)\n\tat org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:979)\n\tat org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1014)\n\tat org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:914)\n\tat jakarta.servlet.http.HttpServlet.service(HttpServlet.java:590)\n\tat org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:885)\n\tat jakarta.servlet.http.HttpServlet.service(HttpServlet.java:658)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:195)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)\n\tat org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:51)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)\n\tat org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)\n\tat org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)\n\tat org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201)\n\tat org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:116)\n\tat org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:164)\n\tat org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:140)\n\tat org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:167)\n\tat org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:90)\n\tat org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:483)\n\tat org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:116)\n\tat org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:93)\n\tat org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:74)\n\tat org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:344)\n\tat org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:398)\n\tat org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:63)\n\tat org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:903)\n\tat org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1769)\n\tat org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:52)\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1189)\n\tat org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:658)\n\tat org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:63)\n\tat java.base/java.lang.Thread.run(Thread.java:1583)\n",
  "message": "Validation failed for object='userRequestDto'. Error count: 1",
  "errors": [
    {
      "cause": {
        "objectName": "userRequestDto",
        "field": "cpf",
        "rejectedValue": "1111111111",
        "codes": [
          "CpfValid.userRequestDto.cpf",
          "CpfValid.cpf",
          "CpfValid.java.lang.String",
          "CpfValid"
        ],
        "arguments": [
          {
            "codes": [
              "userRequestDto.cpf",
              "cpf"
            ],
            "arguments": null,
            "defaultMessage": "cpf",
            "code": "cpf"
          }
        ],
        "defaultMessage": "Cpf inválido.",
        "bindingFailure": false,
        "code": "CpfValid"
      },
      "codes": [
        "CpfValid.userRequestDto.cpf",
        "CpfValid.cpf",
        "CpfValid.java.lang.String",
        "CpfValid"
      ],
      "defaultMessage": "Cpf inválido.",
      "arguments": [
        {
          "codes": [
            "userRequestDto.cpf",
            "cpf"
          ],
          "arguments": null,
          "defaultMessage": "cpf",
          "code": "cpf"
        }
      ]
    }
  ],
  "path": "/user"
}
```

Para detalha melhor essa mensagem de erro, primeiro a criamos uma classe ValidationError que irá herda da classe MessageError:
```java
package com.willians.ListifyShop.exception;  
  
import java.time.Instant;  
import java.util.ArrayList;  
import java.util.List;  
  
public class ValidationError extends MessageError {  
    private List<FieldMesseger> errors = new ArrayList<>();  
  
    public ValidationError() {  
    }  
    public ValidationError(Instant timestamp, Integer status, String error, String message, String path) {  
        super(timestamp, status, error, message, path);  
    }  
  
    public void addError(String fieldName, String message){  
        errors.add(new FieldMesseger(fieldName, message));  
    }  
  
    public List<FieldMesseger> getErrors(){  
        return errors;  
    }  
}
```

Essa classe tem uma lista de FieldMesseger no qual armazena todos os erros da validação

```java
package com.willians.ListifyShop.exception;  
  
public class FieldMesseger {  
    private String fieldName;  
    private String message;  
  
    public FieldMesseger() {}  
  
    public FieldMesseger(String fieldName, String message) {  
        this.fieldName = fieldName;  
        this.message = message;  
    }  
  
    public String getFieldName() {  
        return fieldName;  
    }  
  
    public void setFieldName(String fieldName) {  
        this.fieldName = fieldName;  
    }  
  
    public String getMessage() {  
        return message;  
    }  
  
    public void setMessage(String message) {  
        this.message = message;  
    }  
}
```

Na classe CustomControllerAdvice iremos adicionar o método anotationError MethodArgumentNotValidException

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ValidationError> anotationError(MethodArgumentNotValidException e, HttpServletRequest request){
        int cod = HttpStatus.UNPROCESSABLE_ENTITY.value();
        ValidationError error = new ValidationError(
                Instant.now(),
                cod,
                "Validation exception",
                e.getMessage(),
                request.getRequestURI()
        );

        e.getBindingResult().getFieldErrors().stream().forEach(field -> error.addError(field.getField(), field.getDefaultMessage()));

        return ResponseEntity.status(cod).body(error);
    }
```

No final temos a mensagem de erro menor:

```json
{
  "timestamp": "2025-07-13T20:12:55.207693356Z",
  "status": 422,
  "error": "Validation exception",
  "message": "Validation failed for argument [0] in public com.willians.ListifyShop.dto.UserResponseDto com.willians.ListifyShop.controller.UserController.addUser(com.willians.ListifyShop.dto.UserRequestDto): [Field error in object 'userRequestDto' on field 'cpf': rejected value [1111111111]; codes [CpfValid.userRequestDto.cpf,CpfValid.cpf,CpfValid.java.lang.String,CpfValid]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [userRequestDto.cpf,cpf]; arguments []; default message [cpf]]; default message [Cpf inválido.]] ",
  "path": "/user",
  "errors": [
    {
      "fieldName": "cpf",
      "message": "Cpf inválido."
    }
  ]
}
```

## Referências:
https://wpsilva.medium.com/manipulando-exce%C3%A7%C3%B5es-em-java-com-spring-f9927b52f59b