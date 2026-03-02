## SyncQA Jira Integration SDK

SDK Java pura (sem dependências externas) para integrar seus testes de QA com o Jira, criando casos de teste e reportando resultados de execução via REST API.

### Dependência Maven

Adicione a dependência no seu `pom.xml`:

```xml
<dependency>
    <groupId>io.github.syncqa</groupId>
    <artifactId>jira-integration-sdk</artifactId>
    <version>1.0.1</version>
</dependency>
```

### Dependência Gradle (Kotlin DSL)

```kotlin
dependencies {
    implementation("io.github.syncqa:jira-integration-sdk:1.0.1")
}
```

### Configuração de credenciais

Você pode configurar o acesso ao Jira de duas formas.

- **Via variáveis de ambiente** (modo mais simples):
  - `JIRA_BASE_URL` – URL base do Jira, por exemplo: `https://suaorg.atlassian.net`
  - `JIRA_EMAIL` – e-mail do usuário (conta usada no Jira)
  - `JIRA_API_TOKEN` – token de API gerado no Jira

- **Via builder em código**: informando `url`, `email` e `apiToken` diretamente no código.

### Classe principal da SDK

A fachada principal é a classe `com.sync.qa.sdk.JiraQA`, que expõe operações de alto nível:

- **`createTestCase(TestCase testCase)`**: cria um novo Test Case no Jira.
- **`createTestCaseIfNotExists(TestCase testCase)`**: cria o Test Case apenas se ainda não existir (mesma combinação de projeto + summary).
- **`reportExecution(Execution execution)`**: adiciona um comentário em um issue com o resultado de uma execução de testes.
- **`isConfigured()`**: verifica se a autenticação está corretamente configurada.

Os modelos de domínio principais são:

- `com.sync.qa.sdk.domain.TestCase`
+- `com.sync.qa.sdk.domain.Execution`

---

## Exemplo de projeto de consumo (Maven)

Abaixo um exemplo mínimo de projeto Maven que consome a SDK.

### Estrutura de pastas

```text
meu-projeto-jira/
├─ pom.xml
└─ src/
   └─ main/
      └─ java/
         └─ com/
            └─ exemplo/
               └─ jira/
                  └─ DemoJiraIntegrationApplication.java
```

### `pom.xml` do projeto exemplo

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.exemplo</groupId>
    <artifactId>meu-projeto-jira</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- SDK de integração com Jira -->
        <dependency>
            <groupId>io.github.syncqa</groupId>
            <artifactId>jira-integration-sdk</artifactId>
            <version>1.0.1</version>
        </dependency>
    </dependencies>

</project>
```

### `DemoJiraIntegrationApplication.java`

Exemplo simples criando um Test Case e reportando uma execução.

```java
package com.exemplo.jira;

import com.sync.qa.sdk.JiraQA;
import com.sync.qa.sdk.domain.Execution;
import com.sync.qa.sdk.domain.TestCase;

import java.util.Optional;

public class DemoJiraIntegrationApplication {

    public static void main(String[] args) {
        // 1) Construindo a fachada JiraQA a partir de variáveis de ambiente
        JiraQA jira = JiraQA.fromEnvironment();

        if (!jira.isConfigured()) {
            System.out.println("JiraQA não configurado. Verifique as variáveis de ambiente.");
            return;
        }

        // 2) Criando um Test Case
        TestCase tc = new TestCase();
        tc.setProjectKey("QA"); // chave do projeto no Jira
        tc.setSummary("Exemplo de caso de teste criado via SDK");
        tc.setDescription("Descrição longa do caso de teste...");

        Optional<String> maybeKey = jira.createTestCaseIfNotExists(tc);
        String issueKey = maybeKey.orElse(null);

        if (issueKey == null) {
            System.out.println("Não foi possível criar/obter o Test Case.");
            return;
        }

        System.out.println("Test Case no Jira: " + issueKey);

        // 3) Reportando o resultado de uma execução de testes
        Execution execution = new Execution();
        execution.setIssueKey(issueKey);
        execution.setStatus("PASSED"); // ou FAILED, ERROR, etc. (campo livre)
        execution.setTotal(10);
        execution.setPassed(10);
        execution.setFailed(0);
        execution.setErrors(0);
        execution.setRunUrl("https://meu-ci-server/job/123"); // link opcional para o pipeline

        boolean reported = jira.reportExecution(execution);
        System.out.println("Execução reportada? " + reported);
    }
}
```

### Executando o exemplo

1. Configure as variáveis de ambiente `JIRA_BASE_URL`, `JIRA_EMAIL` e `JIRA_API_TOKEN`.
2. No diretório do projeto exemplo, execute:

```bash
mvn compile exec:java -Dexec.mainClass="com.exemplo.jira.DemoJiraIntegrationApplication"
```

3. Verifique no Jira:
   - O novo issue/caso de teste criado.
   - O comentário com o resumo da execução dos testes.

---

## Uso avançado (builder explícito)

Se preferir não usar variáveis de ambiente, você pode construir a instância de `JiraQA` manualmente:

```java
JiraQA jira = JiraQA.builder()
    .url("https://suaorg.atlassian.net")
    .email("seu-email@dominio.com")
    .apiToken("seu-token-api")
    .issueType("Test Case") // opcional: tipo de issue customizado
    .build();
```

O restante do fluxo (`TestCase`, `Execution` e chamadas de API) permanece exatamente igual ao exemplo anterior.


