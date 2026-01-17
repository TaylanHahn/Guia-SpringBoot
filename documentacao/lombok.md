> Guia de Referência 📚 
## LOMBOK 

O **Project Lombok** é uma das bibliotecas mais populares no ecossistema Java/Spring porque ataca um dos maiores pontos de dor da linguagem: a verbosidade (o famoso "boilerplate code").

Abaixo, um guia focado para usar o Lombok de forma profissional em seus projetos Spring Boot.

**Objetivo:** Reduzir código repetitivo (Getters, Setters, Construtores, Logs) através de anotações que geram esse código automaticamente em tempo de compilação.

## 1. Instalação e Configuração
Para que o Lombok funcione, você precisa de duas coisas:

- 1. Dependência no projeto (Maven ou Gradle).
- 2. Plugin na sua IDE (IntelliJ, Eclipse, VS Code). Sem o plugin, a IDE marcará seu código como erro, pois ela não "vê" os métodos gerados.

Dependência (Maven):
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
````
 - *Se usar o Spring Initializr, basta selecionar "Lombok" na lista de dependências.*

## 2. As Anotações Essenciais
A "Santíssima Trindade" (Getters, Setters e ToString)

Em vez de gerar 50 linhas de código para uma classe simples:

```java
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class ClienteDTO {
    private String nome;
    private String email;
    // O Lombok gera getNome(), setNome(), getEmail(), setEmail() e toString()
}
````


