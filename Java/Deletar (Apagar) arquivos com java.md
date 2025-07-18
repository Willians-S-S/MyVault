```java
public ResponseEntity<Void> deleteImg(String nameImg){  
     try {  
         Path imgPath = Paths.get(this.path).resolve(nameImg);  
         Files.deleteIfExists(imgPath);  
  
         return  ResponseEntity.ok().build();  
     } catch (IOException e) {  
         throw new RuntimeException(e);  
     }  
}
```

### Visão Geral

O método `deleteImg` é uma função que permite **excluir uma imagem** específica do diretório de armazenamento. Ele recebe o nome da imagem como parâmetro, localiza o arquivo e o remove, se ele existir.

### Parâmetros

- **`String nameImg`**: O nome do arquivo da imagem a ser excluída.
    

### Retorno

- **`ResponseEntity<Void>`**: Retorna um `ResponseEntity` vazio com status HTTP `200 OK` em caso de sucesso na exclusão (ou se o arquivo não existia).
    

### Funcionamento Simplificado

1. **Caminho do Arquivo**: Constrói o caminho completo para a imagem, combinando o diretório de armazenamento (`this.path`) com o `nameImg` fornecido.
    
2. **Exclusão Segura**: Tenta **deletar o arquivo** no caminho especificado usando `Files.deleteIfExists()`. Essa função é segura, pois não lança um erro se o arquivo já não existir.
    
3. **Sucesso**: Em caso de sucesso na exclusão (ou se o arquivo já não existia), retorna um status HTTP `200 OK`.
    
4. **Erro**: Se ocorrer uma `IOException` durante a tentativa de exclusão (por exemplo, problemas de permissão), a exceção é capturada e relançada como uma `RuntimeException`.
    

---

## Referências
https://codegym.cc/pt/groups/posts/pt.341.excluir-um-arquivo-em-java