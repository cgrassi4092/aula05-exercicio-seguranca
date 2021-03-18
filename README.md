# Aula 05 - Exercício: Adicionando segurança ao nosso pipeline

Para o nosso exercício vamos usar uma aplicação de código aberto, propositalmente vulnerável, a [WebGoat](https://owasp.org/www-project-webgoat/)

## 1. Preparação do exercício

Para executar o exercício do começo ao fim vamos precisar configurar contas pessoais em algumas plataformas e ter um fork do [WebGoat](https://owasp.org/www-project-webgoat/) no nosso GitHub pessoal.

### 1.1 Plataformas

Você deve ter um perfil/conta criado nas seguintes plataformas/serviços

1. [Sonarcloud](https://sonarcloud.io/) - De preferência, faça o login/cadastro com sua conta do GitHub
2. [Snyk](https://snyk.io/) - De preferência, faça o login/cadastro com sua conta do GitHub
3. [GitGuardian](https://www.gitguardian.com/) - De preferência, faça o login/cadastro com sua conta do GitHub

### 1.2 WebGoat

Faça um fork do [WebGoat](https://github.com/WebGoat/WebGoat) para sua conta pessoal. Não é necessário baixar o código na sua máquina pessoal. Vamos rodar tudo online 🎉

### 1.3 Configuração do repositório

No seu fork, vá em `Setings` > `Security & analysis` e clique em `Enable` nas três opções disponíveis.

Em seguida, no menu da esquerda, vá em `Options`, na seção `Features` e marque o checkbox `Issues`.

## 2. Configuração das Análises de Segurança

### 2.1 Sonarcloud

1. Acesse <https://sonarcloud.io/>, faça login com sua conta do GitHub e, em seguida clique no ícone ➕ no canto superior direito, próximo à barra de busca.
2. Selecione a opção `Analyze new project`
3. Clique em `Import another organization`
4. Clique no botão `Choose an organization on GitHub`
5. Clique no seu usuário
6. Selecione `All repositories` e clique em `Install`
7. Na tela seguinte, clique `Continue`
8. Selecione `Free plan` e clique em `Create organization`
9. Na lista de repositórios procure e selecione `WebGoat`. Depois clique em `Set Up`

O Sonarcloud irá informar que não recomenda uma análise automática do repositório. Assim, vamos criar uma Action para executar a análise.

11. Clique em `With GitHub Actions` e siga o passo a passo informado na tela.
    1. Quando perguntado _**What option best describes your build?**_, selecione `Maven`

:rotating_light Você precisará alterar algumas pequenas configurações para a executar o projeto:

1. As configurações do seu `pom.xml` devem ser ajustadas para entender que o [WebGoat](https://owasp.org/www-project-webgoat/) possui vários módulos. Ajuste conforme o snippet de código abaixo, substituindo o valor em `<sonar.projectKey>` pela chave correta do seu projeto no Sonarcloud.

```xml
<!-- Sonar properties-->
<sonar.projectKey>pedrolacerda_WebGoat</sonar.projectKey>
<sonar.moduleKey>${artifactId}</sonar.moduleKey>
<sonar.organization>pedrolacerda</sonar.organization>
<sonar.host.url>https://sonarcloud.io</sonar.host.url>
```

2. A última linha do arquivo de workflow do Sonar que você criou, na qual o projeto é executado, deve ser alterada para ficar assim: 

```yml
run: ./mvnw verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar
```

Segue exemplo de arquivo completo

```yml
name: Sonarcloud
on:
  workflow_dispatch:
  pull_request:
    branches: [develop]
  
jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
          
      - name: Set up JDK 11
        uses: actions/setup-java@v1
        with:
          java-version: 11
          
      - name: Cache SonarCloud packages
        uses: actions/cache@v1
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
          
      - name: Cache Maven packages
        uses: actions/cache@v1
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2

      - name: Build and analyze
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Needed to get PR information, if any
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: ./mvnw verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar

```

https://github.com/marketplace/actions/owasp-zap-full-scan
https://www.zaproxy.org/docs/
