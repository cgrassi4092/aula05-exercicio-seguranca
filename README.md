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

Faça um fork do [WebGoat](https://github.com/WebGoat/WebGoat) para sua conta pessoal. Não é necessá

Enable Advanced Security

Enable dependabot alerts

Enable issues

mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar

```xml
<!-- Sonar properties-->
<sonar.projectKey>pedrolacerda_WebGoat</sonar.projectKey>
<sonar.moduleKey>${artifactId}</sonar.moduleKey>
<sonar.organization>pedrolacerda</sonar.organization>
<sonar.host.url>https://sonarcloud.io</sonar.host.url>
```

Add snyk


https://github.com/marketplace/actions/owasp-zap-full-scan
https://www.zaproxy.org/docs/

GitGuardian
