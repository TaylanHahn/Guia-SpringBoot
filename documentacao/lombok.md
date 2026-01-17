> Guia de Referência 📚 
## LOMBOK 🧩☕

O **Project Lombok** é uma das bibliotecas mais populares no ecossistema **Java / Spring** porque ataca um dos maiores pontos de dor da linguagem: a **verbosidade** (o famoso *boilerplate code*).

Abaixo, um **guia focado** para usar o Lombok de forma **profissional** em seus projetos **Spring Boot**.

**🎯 Objetivo:**  
Reduzir código repetitivo (Getters, Setters, Construtores, Logs) através de **anotações** que geram esse código automaticamente **em tempo de compilação** ⚙️.

---

## 1. Instalação e Configuração 🧩

Para que o Lombok funcione corretamente, você precisa de **duas coisas essenciais** ✅:

- 📦 **1. Dependência no projeto** *(Maven ou Gradle)*.
- 🛠️ **2. Plugin na sua IDE** *(IntelliJ, Eclipse, VS Code)*.  
  ⚠️ Sem o plugin, a IDE marcará seu código como erro, pois ela não “vê” 👀 os métodos gerados automaticamente pelo Lombok.


Dependência (Maven):
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
````
 - *Se usar o Spring Initializr, basta selecionar "Lombok" na lista de dependências.*

## 2. As Anotações Essenciais 🏷️
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

### Construtores
Essenciais para injeção de dependência e imutabilidade.

| Anotação                   | O que gera?                                         | Uso comum                                                   |
|---------------------------|-----------------------------------------------------|-------------------------------------------------------------|
| `@NoArgsConstructor`      | Construtor vazio: `public Cliente() {}`             | Obrigatório para JPA / Hibernate (`@Entity`).               |
| `@AllArgsConstructor`     | Construtor com todos os campos.                      | Testes unitários ou Builders manuais.                       |
| `@RequiredArgsConstructor`| Construtor com campos `final` ou `@NonNull`.        | ⭐ Ouro: Injeção de dependência no Spring.                  |

### `@Data`
- **O que faz?** Agrupa `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode` e `@RequiredArgsConstructor`.
- ✅ **Quando usar:** Em DTOs e classes de modelo simples.
- ❌ **Quando NÃO usar:** Em Entidades JPA (`@Entity`). Veremos o motivo nas Boas Práticas.

````java
@Data
public class ProdutoDTO {
    private Long id;
    private String descricao;
    private BigDecimal preco;
}
````

### `@Builder`
Permite criar objetos de forma fluente, sem precisar de construtores gigantes com 10 parâmetros.
````java
@Builder
public class Pedido {
    private Long id;
    private String cliente;
    private BigDecimal total;
}

// Uso:
Pedido p = Pedido.builder()
    .cliente("Ana")
    .total(new BigDecimal("100.00"))
    .build();
````

### `@Slf4j` (logging)
Evita ter que declarar private static final Logger log = ... em toda classe.
````java
@Service
@Slf4j // Cria automaticamente uma variável 'log'
public class PagamentoService {
    public void processar() {
        log.info("Processando pagamento...");
        try {
            // lógica
        } catch (Exception e) {
            log.error("Erro ao processar", e);
        }
    }
}
````

## 3. Integração com Spring Boot 🌱🔌
Esta é a dica mais valiosa deste guia. Use o Lombok para eliminar a anotação `@Autowired`.

**Field Injection (Antigo) — ❌ Não recomendado**:
````java
@Service
public class UsuarioService {
    @Autowired // Difícil de testar, esconde dependências
    private UsuarioRepository repository;
}
````

**Constructor Injection (com Lombok) — ✅ Recomendado**
Ao declarar os campos como final e usar `@RequiredArgsConstructor`, o Lombok cria um construtor com esses campos. O Spring, ao ver um único construtor, injeta as dependências automaticamente.
````java
@Service
@RequiredArgsConstructor // Gera construtor com 'repository'
public class UsuarioService {

    private final UsuarioRepository repository; // O campo deve ser final

    public void salvar(Usuario u) {
        repository.save(u);
    }
}
````

## 4. Boas Práticas e "Pegadinhas" 🧠⚠️
Cuidado com JPA e `@Data`
Nunca use `@Data` em classes `@Entity`.

- ⚠️ **Problema 1 (Loop Infinito):** O `@ToString` gera uma String com todos os campos. Se você tiver um relacionamento bidirecional (ex: Pedido <-> Itens), o toString do Pedido chama o do Item, que chama o do Pedido... gerando StackOverflowError.

- ⚠️ **Problema 2 (Performance):** O `@EqualsAndHashCode` pode tentar carregar dados Lazy (preguiçosos) do banco para comparar objetos, causando lentidão severa.

**Solução para Entidades:** Use anotações separadas e exclua relacionamentos perigosos.
````java
@Entity
@Getter
@Setter
@NoArgsConstructor // Obrigatório para JPA
@AllArgsConstructor
public class Pedido {
    
    @Id
    private Long id;

    // Exclua relacionamentos do ToString
    @ToString.Exclude 
    @OneToMany(mappedBy = "pedido")
    private List<Item> itens;
}
````

### Imutabilidade com `@Value` 🪨
Se você quer criar um DTO que é imutável (ninguém muda os dados depois de criado), use `@Value` em vez de `@Data`. Ele torna todos os campos private final e não gera Setters.

| Recurso / Anotação              | Spring Boot | JPA (Entidades) | DTOs |
|---------------------------------|-------------|------------------|------|
| Getters / Setters               | ✅          | ✅               | ✅   |
| `@Data`                         | ⚠️          | ❌ *(Perigo)*    | ✅ *(Ideal)* |
| `@Builder`                      | ✅          | ⚠️ *(Cuidado com construtores)* | ✅ |
| `@RequiredArgsConstructor`      | ✅ *(Para Injeção)* | ❌ | ❌ |
| `@Slf4j`                        | ✅ *(Services / Controllers)* | ❌ | ❌ |

