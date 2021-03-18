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

3. Execute o workflow manualmente e aguarde os resultados aparecerem no Sonarcloud. 
   1. :question: O que mais te chamou atenção?

### 2.1 GitHub Advanced Security

1. Clique na aba `Security` e em seguida, no menu esquerdo, `Code scanning alerts`
2. Na tela seguinte, no canto direto, clique em `Set up more code scanning tools`
3. A primeira opção é o **CodeQL Analysis**, clique em `Set up this workflow`
4. Substitua o conteúdo do arquivo de workflow pelo código abaixo:

```yml
name: "CodeQL"

on:
  workflow_dispatch:

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        language: [ 'java', 'javascript' ]

    steps:

    - name: Checkout repository
      uses: actions/checkout@v2
      
    - name: Initialize CodeQL
      uses: github/codeql-action/init@v1
      with:
        languages: ${{ matrix.language }}
    
    - name: Set up JDK 1.11
      if: matrix.language == 'java'
      uses: actions/setup-java@v1
      with:
        java-version: 1.11
      
    - name: Build with Maven
      if: matrix.language == 'java'
      run: mvn -B package --file pom.xml

    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v1

```

5. Execute o workflow manualmente e aguarde os resultados aparecerem em `Security` > `Code scanning alerts` > `CodeQL`
   1. :question: Os resultados parecem com os do Sonarcloud?

### 2.1 Anchore

O [Anchore](https://anchore.com/) é uma ferramenta open source para análise de vulnerabilidades. Vamos configurá-lo para analisar a segurança do container do WebGoat.

1. Em `Security` > `Code scanning alerts`, selecione novamente `Set up more code scanning tools`
2. A opção na segunda coluna da primeira linha é **Anchore Container Scan**. Clique em `Set up this workflow`
3. Elimine as linhas **24** e **25** e altere o parâmetro `image` para: 

```yml
image: "webgoat/goatandwolf"
```

O arquivo final deve ficar como o a seguir:

```yml
name: Anchore Container Scan

on:
  workflow_dispatch:

jobs:

  Anchore-Build-Scan:
  
    runs-on: ubuntu-latest
    steps:
    
    - name: Checkout the code
      uses: actions/checkout@v2
      
    - name: Run the Anchore scan action itself with GitHub Advanced Security code scanning integration enabled
      uses: anchore/scan-action@main
      with:
        image: "webgoat/goatandwolf"
        acs-report-enable: true
        fail-build: false
        
    - name: Upload Anchore Scan Report
      uses: github/codeql-action/upload-sarif@v1
      with:
        sarif_file: results.sarif
```

4. Execute o workflow manualmente e verifique os resultados na aba `Security` > `Code scanning alerts` > `Anchore Container Vulnerability Report (T0)`
   1. :question: Containers são seguros?

## 3. Análise de dependências

Você deve ter percebido que alguns pull requests foram abertos automaticamente no seu repositório. Além disso, há um banner de alerta no repositório:

![image](https://user-images.githubusercontent.com/609076/111558661-f960fc80-876d-11eb-9735-0c1373c9f5c6.png)

:question: O que são esses alertas?
:question: Por que os PRs foram abertos?

## 4. Shifting left na segurança

Vimos até agora como realizar análises no código que está no nosso repositório no branch principal. Mas e se quisermos validar que estamos entregando código seguro antes que alguma vulnerabilidade seja entregue em produção?

Vamos criar um PR para ver como isso ficaria :+1:

1. Crie um novo branch chamado `new-feature-branch`
2. Crie uma nova pasta na raíz do projeto chamada `js` e adicione os seguintes arquivos:
   1. [index.js](https://github.com/pedrolacerda/WebGoat/blob/new-feature-branch/js/index.js)
   2. [insecurity.js](https://github.com/pedrolacerda/WebGoat/blob/new-feature-branch/js/insecurity.js)
   3. [secrets.js](https://github.com/pedrolacerda/WebGoat/blob/new-feature-branch/js/secrets.js)
   4. [userApiSpec.js](https://github.com/pedrolacerda/WebGoat/blob/new-feature-branch/js/userApiSpec.js)
3. Em seguida, clique em `Pull requests` e verá que apareceu um banner amarelo sugerindo que você abra um novo PR para o merge das modificações. Clique no botão `Compare & pull request`
4. Tenha certeza que você está abrindo um pull request apenas para dentro do seu próprio repositório e clique `Create pull request`

:question: Como os workflows de segurança foram executados?

## 5. "Desafio"

Configure o [GitGuardian](https://www.gitguardian.com/) e o [Snyk](https://snyk.io/) para analisarem o repositório.

:question: Qual tipo de análise essas ferramentas fazem?

## 6. Extra

[ZAP Security testing](https://www.zaproxy.org/)

O Zap permite executar **DAST** via GitHub Actions. Mais informações em: <https://github.com/marketplace/actions/owasp-zap-full-scan>
