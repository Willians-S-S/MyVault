
### Passo 1: Criar a Anotação Personalizada
Primeiro criamos a interface que será nossa anotação

```java
package com.willians.ListifyShop.validation;  
  
import jakarta.validation.Constraint;  
import jakarta.validation.Payload;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
@Constraint(validatedBy = CpfValidator.class)  
@Target({ElementType.FIELD})  
@Retention(RetentionPolicy.RUNTIME)  
public @interface CpfValid {  
    String message() default "Cpf inválido.";  // Mensagem padrão de erro
    Class<?>[] groups() default {};  // Grupos de validação (Opcional) 
    Class<? extends Payload>[] payload() default {};  // Payload para metadados (Opcional)
}

```

@Constraint(validatedBy = CpfValidator.class)  // Define a classe que implementa a validação
@Target({ ElementType.FIELD }) // A anotação pode ser usada apenas em campos
@Retention(RetentionPolicy.RUNTIME) // A anotação estará disponível em tempo de execução

### Passo 2: Implementar a Lógica de Validação
Nessa passo criamos a classe que vai validar as informações

```java
package com.willians.ListifyShop.validation;  
  
import jakarta.validation.ConstraintValidator;  
import jakarta.validation.ConstraintValidatorContext;  
  
public class CpfValidator implements ConstraintValidator<CpfValid, String> {  
  
    @Override  
    public void initialize(CpfValid cpf){}  
  
    @Override  
    public boolean isValid(String value, ConstraintValidatorContext context) {  
        if(!value.matches("[0-9]+") || value.contains(" ") || value.length() != 11){  
            return false;  
        }  
  
        int acumuladorParaDitio1 = 0;  
        int acumuladorParaDitio2 = 0;  
        final int ONZE = 11;  
  
        for (int i = 0; i < 10; i++){  
            int valor = Integer.parseInt(String.valueOf(value.charAt(i)));  
            if(i < 9){  
                acumuladorParaDitio1 +=  valor * (ONZE - (i + 1));  
            }  
            acumuladorParaDitio2 += valor * (ONZE - i);  
        }  
  
        int modulo1 = acumuladorParaDitio1 % ONZE;  
        int modulo2 = acumuladorParaDitio2 % ONZE;  
  
        int resto1 = ONZE - modulo1;  
        int resto2 = ONZE - modulo2;  
  
        Integer digito1 = resto1 < 10 ? resto1 : 0;  
        Integer digito2 = resto2 < 10 ? resto2 : 0;  
  
        Integer digito1Origem = Integer.parseInt(String.valueOf(value.charAt(9)));  
        Integer digito2Origem = Integer.parseInt(String.valueOf(value.charAt(10)));  
  
        if (digito1.equals(digito1Origem) && digito2.equals(digito2Origem)){  
            return true;  
        }  
  
        return false;  
    }  
}
```

### Passo 3: Usar a Anotação
Agora podemos usar nossa anotação em nossas classe, deste entety, dto até record:
```java
package com.willians.ListifyShop.dto;  
  
  
import com.willians.ListifyShop.validation.CpfValid;  
  
public record UserRequestDto(String name, @CpfValid String cpf, String email, String password) {  
}
```

### Passo 4: Testar a Validação
Para testar a validação, criamos um controlador REST que recebe um objeto `UserRequestDto` e valida os dados antes de salvar no banco de dados.

```java
@PostMapping  
public UserResponseDto addUser(@Valid @RequestBody UserRequestDto userRequest){  
    return this.userService.addUser(userRequest);  
}
```
A anotação `@Valid` ativa a validação automática do Bean Validation.

#### Referências:
https://dev.to/uiratan/como-criar-uma-annotation-que-realize-validacao-personalizada-para-garantir-valores-unicos-no-banco-5a51