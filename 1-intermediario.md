# ☕🌱 | Intermediário
Foco: Persistência (JPA), Validação, Tratamento de Erros e Padrões de Projeto (DTO).

## 1. Persistência de Dados (Spring Data JPA) 💾
> Contexto: Backend/Banco de Dados — Uso: Essencial

O *Spring Data JPA* abstrai a complexidade do JDBC e do Hibernate. O foco muda de "escrever SQL" para "gerenciar Entidades".

### Mapeamento Objeto-Relacional (ORM)
Transforma classes Java em tabelas do banco.

| Anotação        | Significado / Função                                   | Quando usar                                                                 | Boas Práticas                                                                                                  |
|-----------------|--------------------------------------------------------|-----------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| ***@Entity***         | Marca a classe como uma entidade do banco de dados.    | Obrigatório para qualquer classe que represente uma tabela.                 | O construtor vazio (padrão) é obrigatório pela especificação JPA.                                                |
| ***@Table***          | Define detalhes da tabela (nome, schema).              | Quando o nome da classe difere do nome da tabela no banco.                  | Use para garantir nomes compatíveis com bancos legados (ex: `tb_usuario`).                                      |
| ***@Id***             | Define a Chave Primária (PK).                           | Obrigatório em toda `@Entity`.                                              | Prefira tipos objetos (`Long`, `UUID`) em vez de primitivos para permitir `null` antes da persistência.        |
| ***@GeneratedValue*** | Define a estratégia de geração do ID.                  | Utilizada em conjunto com `@Id`.                                            | `GenerationType.IDENTITY` é o mais comum para MySQL/PostgreSQL simples.                                          |
| ***@Column***         | Configura a coluna (nome, nullable, length, unique).   | Para impor restrições no banco ou alterar nomes de colunas.                 | Sempre defina `nullable = false` em campos obrigatórios para garantir integridade dos dados.                   |
| ***@Transient***      | Ignora o campo na persistência.                        | Quando o campo existe na classe, mas não deve ser salvo no banco.           | Útil para cálculos em tempo de execução (ex: idade calculada pela data de nascimento).                         |

### Repositórios Inteligentes
Em vez de escrever DAOs manuais, estendemos interfaces.
- **Interface** ➜ JpaRepository<Entidade, TipoID>
- **Conceito (Query Methods)** ➜ O Spring cria o SQL baseado no nome do método.
- `findByEmail(String email)` ⟶ Gera: `SELECT * FROM ... WHERE email = ?`
- `existsByCpf(String cpf)` ⟶ Retorna booleano.

## 2. Validação de Dados (Bean Validation) 🔐
> Contexto: Web/Backend — Uso: Muito Comum

Evite encher seu código de *if (x == null)*. Use anotações declarativas da especificação *Jakarta Validation* (antigo Hibernate Validator).

### Ativação no Controller
Para que as validações funcionem, você deve usar a anotação `@Valid` no parâmetro do método do *Controller*.
````java
public ResponseEntity<?> criar(@RequestBody @Valid UsuarioDTO dto) { ... }
````

### Anotações de Restrição (Dentro do DTO/Entidade)

| Anotação            | Verificação                                             | Tipo de Dado Comum                |
|---------------------|----------------------------------------------------------|-----------------------------------|
| @NotNull            | O valor não pode ser nulo.                               | Qualquer objeto.                  |
| @NotBlank           | Não nulo e pelo menos um caractere não-espaço.           | `String` (melhor opção para textos). |
| @Size               | Tamanho mínimo e/ou máximo.                              | `String`, listas, arrays.         |
| @Min / @Max         | Valor numérico mínimo / máximo.                          | `Integer`, `Long`, `BigDecimal`.  |
| @Email              | Verifica se a string tem formato de e-mail válido.       | `String`.                         |
| @Past / @Future     | Verifica se a data é passada ou futura.                  | `LocalDate`, `LocalDateTime`.     |

- ✅ **Boas Práticas**
  - Coloque mensagens de erro personalizadas: `@NotBlank(message = "O nome é obrigatório")`.
  - Valide nos DTOs, não apenas nas Entidades, para proteger sua API antes mesmo de tentar processar regras de negócio.
 
## 3. Padrão DTO (Data Transfer Object)
> Contexto: Arquitetura — Uso: Boa Prática de Mercado

- 🧠 **Conceito** ➜  Nunca exponha sua @Entity (banco de dados) diretamente no Controller.
- ⚠️ **Problema** ➜ Se você retornar a Entidade, vaza senha, dados internos e gera loops infinitos em relacionamentos bidirecionais (JSON recursivo).
- ✔️ **Solução** ➜ Crie classes POJO simples (DTOs) que representam apenas o que o JSON de entrada/saída deve ter.
- 🔁 **Fluxo Correto:**
  - JSON chega → Controller recebe InputDTO.
  - Conversão InputDTO para Entity.
  - Service processa/salva Entity.
  - Conversão Entity para OutputDTO.

Controller devolve JSON do OutputDTO.
