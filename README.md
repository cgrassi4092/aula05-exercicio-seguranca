# Aula 05 - Exercício: Adicionando segurança ao nosso pipeline

Para o nosso exercício vamos usar uma aplicação de código aberto, propositalmente vulnerável, a [WebGoat](https://owasp.org/www-project-webgoat/)

## 1. Preparação do exercício

Enable dependabot alerts

mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar

```xml
<!-- Sonar properties-->
<sonar.projectKey>pedrolacerda_WebGoat</sonar.projectKey>
<sonar.moduleKey>${artifactId}</sonar.moduleKey>
<sonar.organization>pedrolacerda</sonar.organization>
<sonar.host.url>https://sonarcloud.io</sonar.host.url>
```
