# 🎓 Universidade Federal do Ceará
### Departamento de Computação
### Grupo de Rede de Computadores, Engenharia de Software e Sistemas - GREat

---

## 📚 Disciplina
* **CK0241** - Verificação, Validação e Teste de Software
* **CKP9012** - Verificação e Validação de Software

---

# 🚀 Demo-CI: Aula Prática sobre Servidores de Integração Contínua

---

### 📧 Contato e Equipe

* **Profa. Valéria Lelli Leitão Dantas**
    * Email: valerialelli@ufc.br
* **Monitor: Francisco Jean da Silva de Sousa**
    * Email: franciscojean@alu.ufc.br

Este repositório descreve um roteiro prático para configuração e uso de um **Servidor de Integração Contínua**. O objetivo é proporcionar ao aluno um primeiro contato real com essa prática de desenvolvimento de software.

Se você ainda não sabe o que é **Integração Contínua** e também não entende o papel desempenhado por servidores de CI, recomendamos antes ler o [Capítulo 10](https://engsoftmoderna.info/cap10.html) do nosso livro texto ([Engenharia de Software Moderna](https://engsoftmoderna.info/cap10.html)).

Apesar de existirem diversos servidores de integração contínua, neste roteiro iremos usar um recurso nativo do GitHub, chamado **GitHub Actions**, para configurar um servidor de CI.

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/109080916-232f8200-76e0-11eb-8d02-9ca9f518cea2.png" />
</p>

O Github Actions permite executar programas externos assim que determinados eventos forem detectados em um repositório GitHub. Como nosso intuito é configurar um servidor CI, iremos usar o GitHub Actions para compilar todo o código (_build_) do projeto e rodar seus testes de unidade quando um Pull Request (PR) for aberto no repositório, conforme ilustrado a seguir.

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/110053018-50f77500-7d37-11eb-9ca1-3de609b93584.png" />
</p>

## Programa de Exemplo

Para ilustrar o uso do servidor de CI, vamos usar um programa Java muito simples, que já foi criado e está disponível neste mesmo repositório ([Calculadora.java](https://github.com/aserg-ufmg/demo-ci/blob/main/src/main/java/br/ufmg/dcc/Calculadora.java)):

```java
public class Calculadora {

  public int soma(int x, int y) {
    return x + y;
  }

  public int subtrai(int x, int y) {
    return x - y;
  }
}
```

Quando chegar um PR no repositório, o servidor de CI vai automaticamente realizar um _build_ desse programa e rodar o seguinte teste de unidade (também já disponível no repositório, veja em [CalculadoraTest.java](https://github.com/aserg-ufmg/demo-ci/blob/main/src/test/java/br/ufmg/dcc/CalculadoraTest.java)):

```java
public class CalculadoraTest {

  @Test
  public void testeSoma1() {
    Calculadora calc = new Calculadora();
    int resultadoEsperado = 5;
    int resultadoRetornado = calc.soma(2,3);
    assertEquals(resultadoEsperado, resultadoRetornado);
  }

  @Test
  public void testeSoma2() {
    Calculadora calc = new Calculadora();
    assertEquals(10, calc.soma(4,6));
  }
}
```

## Tarefa #1: Configurar o GitHub Actions

#### Passo 1

Antes de mais nada realize um fork deste repositório. Para isso, basta clicar no botão **Fork** no canto superior direito desta página.

Ou seja, você irá configurar um servidor de CI na sua própria cópia do repositório.

#### Passo 2

Clone o repositório para sua máquina local, usando o seguinte comando (onde `<USER>` deve ser substituído pelo seu usuário no GitHub):

```bash
git clone https://github.com/<USER>/demo-ci.git
```

Em seguida, copie o código a seguir para um arquivo com o seguinte nome: `.github/workflows/actions.yaml`. Isto é, crie diretórios `.github` e depois `workflows` e salve o código abaixo no arquivo `actions.yaml`.

```yaml
name: Github CI
on:
  # Configura servidor de CI para executar o pipeline de tarefas abaixo (jobs) quando 
  # um push ou pull request for realizado tendo como alvo a branch main
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  pipeline:
    runs-on: ubuntu-latest # Os comandos serão executados em um sistema operacional Linux

    steps:
      - name: Git Checkout
        uses: actions/checkout@v2 # Faz o checkout do código recebido

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1 # Configura o Java 1.8
        with:
          java-version: 1.8

      - name: Build
        run: mvn package -Dmaven.test.skip=true # Compila o código fonte

      - name: Unit Test
        run: mvn test # Executada os testes de unidade
```

Esse arquivo ativa e configura o GitHub Actions para -- toda vez que ocorrer um evento `push` ou `pull_request` tendo como alvo a branch principal do repositório -- realizar três tarefas (jobs):

- realizar o checkout do código;
- realizar um build;
- rodar os testes de unidade.

#### Passo 3

Realize um `commit` e um `git push`, isto é:

```bash
git add --all
git commit -m "Configurando GitHub Actions"
git push origin main
```

#### Passo 4

Quando o `push` chegar no repositório principal, o GitHub Actions iniciará automaticamente o fluxo de tarefas configurado no arquivo `actions.yaml` (isto é, build + testes).

Você pode acompanhar o status dessa execução clicando na aba Actions do seu repositório.

<p align="center">
    <img width="80%" src="https://user-images.githubusercontent.com/7620947/110059807-b8b3bd00-7d43-11eb-9e57-e6ba1fa3457a.png" />
</p>

## Tarefa #2: Criando um PR com bug

Para finalizar, vamos introduzir um pequeno bug no programa de exemplo e enviar um PR, para mostrar que ele será "barrado" pelo processo de integração (isto, o nosso teste vai "detectar" o bug e falhar).

#### Passo 1

Introduza um pequeno bug na função `soma` do arquivo [src/main/java/br/ufmg/dcc/Calculadora.java](https://github.com/rodrigo-brito/roteiro-github-actions/blob/main/src/main/java/br/ufmg/dcc/Calculadora.java). Por exemplo, basta alterar a linha 6, alterando o retorno da função para `x + y + 1`, como apresentado abaixo.

```diff
--- a/src/main/java/br/ufmg/dcc/Calculadora.java
+++ b/src/main/java/br/ufmg/dcc/Calculadora.java
@@ -3,7 +3,7 @@ package br.ufmg.dcc;
 public class Calculadora {

   public int soma(int x, int y) {
-    return x + y;
+    return x + y + 1;
   }

   public int subtrai(int x, int y) {
```

#### Passo 2

Após modificar o código, você deve criar um novo branch, realizar um `commit` e `push`:

```bash
git checkout -b bug
git add --all
git commit -m "Incluindo alterações na função soma"
git push origin bug
```

#### Passo 3

Em seguida, crie um Pull Request (PR) com sua modificação. Para isso, basta acessar a seguinte URL em seu navegador: `https://github.com/<USER>/demo-ci/compare/main...bug`, onde `<USER>` deve ser substituido pelo seu usuário no GitHub. Nessa janela, você pode conferir as modificações feitas e incluir uma pequena descrição no PR.

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/111704705-5b793a80-881e-11eb-8422-22d51bde6b19.png" />
</p>

Após finalizar a criação do Pull Request, será iniciada nossa pipeline, ou seja, o próprio GitHub vai fazer o build do sistema e rodar seus testes (como na tarefa #1). Porém, dessa vez os testes não vão passar, como mostrado abaixo:

<p align="center">
    <img width="70%" src="https://user-images.githubusercontent.com/7620947/111704932-a85d1100-881e-11eb-8d3b-31f34bafa986.png" />
</p>

**RESUMINDO**: O Servidor de CI conseguiu alertar, de forma automática, tanto o autor do PR como o integrador de que existe um problema no código submetido, o que impede que ele seja integrado no branch principal do repositório.

## Créditos

Este roteiro foi elaborado por **Rodrigo Brito**, aluno de mestrado do DCC/UFMG, como parte das suas atividades na disciplina Estágio em Docência, cursada em 2020/2, sob orientação do **Prof. Marco Tulio Valente**.
