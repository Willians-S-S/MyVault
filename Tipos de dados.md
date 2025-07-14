Em Java temos os tipos primitivos e tipos de referência que são os primitivos em forma de classe, que possuem métodos.
Em Java, os **tipos de dados** são classificados em **tipos primitivos** e **tipos de referência**. Abaixo estão os tipos mais 
comuns, exemplos de uso e explicações:

---

### 🔹 **Tipos Primitivos (Fundamentais)**
São os tipos básicos que Java oferece diretamente. Cada um tem um tamanho fixo e é armazenado diretamente na memória.

| Tipo Primitivo | Tamanho (em bytes) | Descrição |
|----------------|---------------------|-----------|
| `byte`         | 1                   | Número inteiro assinado de 8 bits. |
| `short`        | 2                   | Número inteiro assinado de 16 bits. |
| `int`          | 4                   | Número inteiro assinado de 32 bits. |
| `long`         | 8                   | Número inteiro assinado de 64 bits. |
| `float`        | 4                   | Número de ponto flutuante de 32 bits. |
| `double`       | 8                   | Número de ponto flutuante de 64 bits. |
| `boolean`      | 1                   | Valor lógico: `true` ou `false`. |
| `char`         | 2                   | Caractere Unicode (ex: 'A', 'ñ'). |

#### Exemplos:
```java
byte byteVar = 10;
short shortVar = 1000;
int intVar = 1000000;
long longVar = 1000000000L;
float floatVar = 3.14f;
double doubleVar = 3.1415926535;
boolean boolVar = true;
char charVar = 'A';
```

---

### 🔹 **Tipos de Referência**
São objetos que são armazenados em endereços de memória. Incluem classes, interfaces, arrays, e objetos criados com `new`.

#### Exemplos:
1. **String** (classe de texto):
```java
String str = "Olá, mundo!";
System.out.println(str.length()); // Imprime 11
```

2. **Objeto de classe**:
```java
class Pessoa {
    String nome;
    int idade;
    public Pessoa(String nome, int idade) {
        this.nome = nome;
        this.idade = idade;
    }
}

Pessoa pessoa = new Pessoa("João", 30);
System.out.println(pessoa.nome); // Imprime "João"
```

3. **Array** (array de objetos):
```java
int[] arrayInt = {1, 2, 3, 4};
System.out.println(arrayInt[0]); // Imprime 1
```

4. **Objeto de classe genérica**:
```java
List<String> list = new ArrayList<>();
list.add("Java");
System.out.println(list.get(0)); // Imprime "Java"
```

---

### 🔹 **Tipos de Wrapper (Classe de Tipos Primitivos)**
Java oferece classes para embalar os tipos primitivos em objetos. São úteis para usar com estruturas que exigem objetos (como 
`List`, `Map`, etc.).

| Tipo Primitivo | Classe de Wrapper |
|----------------|-------------------|
| `byte`         | `Byte`            |
| `short`        | `Short`           |
| `int`          | `Integer`         |
| `long`         | `Long`            |
| `float`        | `Float`           |
| `double`       | `Double`          |
| `boolean`      | `Boolean`         |
| `char`         | `Character`       |

#### Exemplo:
```java
Integer integerVar = 100;
Double doubleVar = 3.14;
Boolean boolVar = true;
```

---

### 🔹 **Diferença entre Tipos Primitivos e Referência**
| Característica         | Tipo Primitivo                | Tipo de Referência              |
|------------------------|-------------------------------|---------------------------------|
| Armazenamento          | Direto na memória             | Endereço de memória             |
| Criação                | Não requer `new`             | Requer `new`                   |
| Necessidade de Boxing | Não necessário                | Necessário (ex: `Integer.valueOf()`) |
| Uso em Coleções        | Não diretamente               | Sim (ex: `List<String>`)        |

---

### 🔹 **Resumo**
- **Tipos primitivos** são fundamentais e otimizados para desempenho.
- **Tipos de referência** são usados para objetos, como `String`, `List`, `Map`, etc.
- **Tipos de wrapper** permitem usar tipos primitivos como objetos (ex: `Integer`).
Em Java, assim como em muitas outras linguagens de programação, os tipos de dados definem o tipo de valor que uma variável pode 
armazenar.  Existem dois tipos principais: **primitivos** e **referência** (ou objetos).

**1. Tipos Primitivos (ou Fundamentais)**

São tipos de dados básicos, que não são objetos. Eles são implementados diretamente na linguagem e ocupam um espaço de memória 
fixo.

*   **`byte`:**
    *   Tamanho: 1 byte (8 bits)
    *   Faixa de valores: -128 a 127
    *   Uso: Armazenar valores inteiros pequenos, como códigos de status HTTP.
    *   Exemplo:
        ```java
        byte idade = 25;
        ```

*   **`short`:**
    *   Tamanho: 2 bytes (16 bits)
    *   Faixa de valores: -32 a 31
    *   Uso: Armazenar valores inteiros pequenos, um pouco maiores que `byte`.
    *   Exemplo:
        ```java
        short contador = 100;
        ```

*   **`int`:**
    *   Tamanho: 4 bytes (32 bits)
    *   Faixa de valores: -2.147.483.648 a 2.147.483.647
    *   Uso: Armazenar valores inteiros de tamanho médio.  É o tipo mais comum para armazenar números inteiros.
    *   Exemplo:
        ```java
        int numero = 1000;
        ```

*   **`long`:**
    *   Tamanho: 8 bytes (64 bits)
    *   Faixa de valores: -9.223.372.036.854.775.808 a 9.223.372.036.854.775.807
    *   Uso: Armazenar valores inteiros grandes, que podem ser maiores que `int`.  Útil quando você precisa representar números 
muito grandes.
    *   Exemplo:
        ```java
        long valorGrande = 123456789012345L; // Adicionar 'L' para indicar que é um long
        ```

*   **`float`:**
    *   Tamanho: 4 bytes (32 bits)
    *   Faixa de valores:  Aproximadamente de -3.402.823.000.000.000.000 a 3.402.823.000.000.000.000
    *   Uso: Armazenar números de ponto flutuante (números com casas decimais).  Menos preciso que `double`.
    *   Exemplo:
        ```java
        float preco = 99.99f; // Adicionar 'f' para indicar que é um float
        ```

*   **`double`:**
    *   Tamanho: 8 bytes (64 bits)
    *   Faixa de valores:  Aproximadamente de -1.797.693.134.897.900.000 a 1.797.693.134.897.900.000
    *   Uso: Armazenar números de ponto flutuante com maior precisão que `float`.  É o tipo mais comum para cálculos 
científicos e financeiros.
    *   Exemplo:
        ```java
        double taxaDeCrescimento = 0.05;
        ```

*   **`boolean`:**
    *   Tamanho: 1 byte (1 bit)
    *   Faixa de valores: `true` ou `false`
    *   Uso: Representar valores lógicos (verdadeiro ou falso).
    *   Exemplo:
        ```java
        boolean estaChovendo = false;
        ```

*   **`char`:**
    *   Tamanho: 2 bytes (16 bits)
    *   Faixa de valores:  Representa um único caractere Unicode.
    *   Uso: Armazenar caracteres individuais, como letras, números, símbolos.
    *   Exemplo:
        ```java
        char letra = 'A';
        ```

**2. Tipos de Referência (ou Objetos)**

São tipos de dados que representam objetos.  Eles armazenam um endereço de memória (um "ponteiro") para o objeto, em vez do 
objeto em si.

*   **`String`:**
    *   Tipo de referência.
    *   Armazena uma sequência de caracteres.
    *   É imutável (não pode ser alterada após a criação).
    *   Exemplo:
        ```java
        String nome = "João";
        ```

*   **`Integer`:**
    *   Tipo de referência.
    *   Armazena um valor inteiro.
    *   É um objeto da classe `Integer`.
    *   Exemplo:
        ```java
        Integer numero = 10;
        ```

*   **`Double`:**
    *   Tipo de referência.
    *   Armazena um valor de ponto flutuante.
    *   É um objeto da classe `Double`.
    *   Exemplo:
        ```java
        Double preco = 99.99;
        ```

*   **`Object`:**
    *   É o tipo de referência mais básico.
    *   Representa qualquer objeto que pode ser criado na linguagem Java.
    *   É o supertipo de todos os outros tipos de referência.
    *   Exemplo:
        ```java
        Object objeto = new String("Olá");
        ```

*   **`Array`:**
    *   Um tipo de referência que armazena uma coleção de elementos do mesmo tipo.
    *   Exemplo:
        ```java
        int[] numeros = new int[5]; // Cria um array de 5 inteiros
        ```

*   **`ArrayList`:**
    *   Um tipo de referência que armazena uma coleção de objetos, implementando a interface `List`.
    *   É uma lista dinâmica, ou seja, o tamanho pode aumentar ou diminuir conforme necessário.
    *   Exemplo:
        ```java
        ArrayList<String> nomes = new ArrayList<>();
        ```

**Como declarar variáveis com tipos:**

```java
int idade = 30;  // Declara uma variável inteira chamada 'idade' e a inicializa com o valor 30.
String nome = "Maria"; // Declara uma variável String chamada 'nome' e a inicializa com o valor "Maria".
double altura = 1.75; // Declara uma variável double chamada 'altura' e a inicializa com o valor 1.75.
```

**Importância de escolher o tipo correto:**

*   **Eficiência:** Usar o tipo de dados correto pode melhorar a eficiência do seu código, especialmente quando se trata de 
cálculos e armazenamento de dados.
*   **Precisão:**  Escolher o tipo de dados apropriado garante que você possa representar os valores com a precisão necessária.
*   **Legibilidade:**  Declarar explicitamente o tipo de uma variável torna o código mais legível e fácil de entender.
*   **Evitar erros:**  Usar tipos de dados incorretos pode levar a erros inesperados e comportamentos imprevisíveis.

**Observações:**

*   Java é uma linguagem fortemente tipada, o que significa que o tipo de uma variável deve ser declarado explicitamente.
*   Você pode usar a palavra-chave `final` para tornar uma variável de referência imutável.
*   Java possui um sistema de tipos que permite que você realize operações em diferentes tipos de dados.