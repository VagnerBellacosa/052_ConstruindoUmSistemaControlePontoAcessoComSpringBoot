# O que é Spring Boot?

[![img](https://secure.gravatar.com/avatar/98def809dd1ff0d88623373ddc725475?s=40&d=mm&r=g)](https://blog.algaworks.com/author/alexandre-afonso/)[Escrito por **Alexandre Afonso**](https://blog.algaworks.com/author/alexandre-afonso/)em 02/02/2017

![img](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/post_o_que_e_spring_boot.png)

Muitas pessoas morrem de medo de terem que configurar uma aplicação do zero. Geralmente, são necessárias várias configurações para só então começar a codificar.

Agora, imagine pular toda essa parte chata de configurações e criar um projeto onde você já tenha tudo o que precisa para começar uma aplicação. Muito bom, não é mesmo?

Se você gostou dessa ideia, continue comigo que eu vou te mostrar como criar esse tipo de aplicação utilizando o **Spring Boot**.

Olha só o que **você irá aprender** nesse artigo:

- O que é Spring Boot
- Instalando o STS (Spring Tool Suite)
- Desenvolver uma pequena aplicação web para listagem de contatos
- Que benefícios tem o DevTools
- Como publicar sua aplicação no Heroku

Vamos lá?

## O que é Spring Boot?



O Spring Boot é um projeto da Spring que veio para facilitar o processo de configuração e publicação de nossas aplicações. A intenção é ter o seu projeto rodando o mais rápido possível e sem complicação.

Ele consegue isso favorecendo a **convenção sobre a configuração**. Basta que você diga pra ele quais módulos deseja utilizar (WEB, Template, Persistência, Segurança, etc.) que ele vai reconhecer e configurar.

Você escolhe os módulos que deseja através dos *starters* que inclui no `pom.xml` do seu projeto. Eles, basicamente, são dependências que agrupam outras dependências. Inclusive, como temos esse grupo de dependências representadas pelo *starter*, nosso `pom.xml` acaba por ficar mais organizado.

Apesar do Spring Boot, através da convenção, já deixar tudo configurado, nada impede que você crie as suas customizações caso sejam necessárias.

O maior benefício do Spring Boot é que ele nos deixa mais livres para pensarmos nas regras de negócio da nossa aplicação.

## Instalando o Spring Tool Suite

O **Spring Tool Suite** é uma IDE baseada em Eclipse que dá algumas facilidades para trabalhos com o Spring no geral. Uma das coisas legais é que ele nos ajuda a criar projetos com **Spring Boot**.

Mas, o STS não é pré-requisito para criação de projetos com Spring Boot. Você vai poder trabalhar com Spring Boot em qualquer IDE que dê suporte ao Maven. Inclusive um site interessante para aqueles que forem utilizar outras IDEs é o [http://start.spring.io](https://start.spring.io/). Ele ajuda na criação de um novo projeto Spring Boot quase que da mesma forma que o STS.

Existem versões do STS para Linux, Mac e Windows. Você pode baixar em: [http://spring.io/tools/sts/all](https://spring.io/tools/sts/all).

Começar um projeto com Spring Boot no STS é bem simples. No menu *File*, selecione *New* e depois *Spring Starter Project*.

A primeira tela que você verá será essa daqui:

[![New Spring Starter Project](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/New-Spring-Starter-Project.png)](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/New-Spring-Starter-Project.png)

Caso você já tenha criado algum projeto Maven no Eclipse, então você deve conhecê-la bem.

O segundo passo que é algo mais específico do Spring Boot. Veja:

[![New Spring Starter Project - Dependencies](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/New-Spring-Starter-Project-Dependencies.png)](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/New-Spring-Starter-Project-Dependencies.png)

A tela acima é onde você escolhe as dependências do seu projeto, ou seja, os módulos que você vai querer utilizar.

Para o projeto aqui do artigo, eu selecionei:

- DevTools (veremos mais a frente)
- JPA
- H2 (nosso banco de dados em memória – não precisamos instalar)
- Web (Spring MVC)
- Thymeleaf (Motor para templates HTML)

Existem vários outros além desses aí. Você pode usar o campo de texto para fazer uma pesquisa e encontrar rapidamente os que desejar.

Basicamente, a única coisa que essa segunda tela faz é permitir a seleção das dependências que ficarão no `pom.xml` do nosso projeto. Nada mais. Mas isso já é de grande ajuda, pois, não precisamos ficar lembrando do *groupId* e do *artifactId* de cada uma.

Depois que fizer a seleção das dependências, você pode clicar em *Finish*. Você vai verificar a seguinte estrutura no seu projeto:

[![Projeto Spring Boot no Package Explorer](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/Projeto-Spring-Boot-no-Package-Explorer.png)](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/Projeto-Spring-Boot-no-Package-Explorer.png)

Repare, na imagem acima, que uma classe importante já foi criada. É a classe `ArtigoSpringBootApplication`:

```
package com.algaworks.contatos;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ArtigoSpringBootApplication {

  public static void main(String[] args) {
    SpringApplication.run(ArtigoSpringBootApplication.class, args);
  }

}
```

Ela contém o método *main* que inicia a nossa aplicação. Engraçado isso, pois, a nossa aplicação será uma aplicação web.

Acontece que o Spring Boot usa um container embarcado (por padrão é o Tomcat) para facilitar o desenvolvimento. Por isso, para iniciar a aplicação, basta executar o método *main* acima.

Nesse ponto, nosso projeto já está pronto para receber as regras de negócio.

## Aplicação para listagem de contatos

Vamos agora criar uma página que vai utilizar o [Spring Data JPA](https://blog.algaworks.com/spring-data-jpa/), o [Spring MVC](https://blog.algaworks.com/spring-mvc/) e o [Thymeleaf](https://blog.algaworks.com/thymeleaf/) para listar contatos vindos da base de dados.

Vou começar criando a entidade `Contato` que vai representar uma tabela do banco de dados:

```
@Entity
public class Contato implements Serializable {

  private static final long serialVersionUID = 1L;
  
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  
  private String nome;
  
  private String email;

  // getters e setters omitidos
}
```

Quanto a camada de persistência, vou utilizar o [Sprint Data JPA](https://blog.algaworks.com/spring-data-jpa/) para criar o meu repositório de contatos. Vou chamá-lo de `Contatos`:

```
public interface Contatos extends JpaRepository<Contato, Long> {
}
```

Detalhe: [não precisamos dar uma implementação para a interface acima](https://blog.algaworks.com/spring-data-jpa/). Isso fica a cargo do Spring Data JPA.

A última classe do projeto será o controlador. Chamarei de `ContatosController`:

```
@Controller
@RequestMapping("/contatos")
public class ContatosController {
  
  @Autowired
  private Contatos contatos;
  
  @GetMapping
  public ModelAndView listar() {
    List&lt;Contato&gt; lista = contatos.findAll();
    
    ModelAndView modelAndView = new ModelAndView("contatos");
    modelAndView.addObject("contatos", lista);
    
    return modelAndView;
  }

}
```

Falta ainda nossa página HTML que, com ajuda do Thymeleaf, vai exibir essa lista de contatos para o usuário. Os arquivos HTML são criados, por padrão, dentro do diretório `src/main/resources/templates`. Então clique com o direito nesse diretório e crie o arquivo `contatos.html`:

```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
  <title>Listagem de contatos utilizando o Spring Boot</title>
</head>

<body>
  <h1>Lista de contatos</h1>
  <table>
    <thead>
      <tr>
        <th>#</th>
        <th>Nome</th>
        <th>E-mail</th>
      </tr>
    </thead>
    <tbody>
      <tr th:each="contato: ${contatos}">  
        <td th:text="${contato.id}"></td>
        <td th:text="${contato.nome}"></td>
        <td th:text="${contato.email}"></td>
      </tr>
    </tbody>
  </table>
</body>
</html>
```

Agora basta você clicar com o direito em cima do projeto, selecionar *Run As*, depois *Spring Boot App* e pronto! Já temos uma aplicação rodando. Acesse *http://localhost:8080/contatos* para conferir.

Caso você não esteja utilizando o STS, então pode rodar o projeto executando o método *main* da classe `ArtigoSpringBootApplication` ou ainda, através do terminal, executando o comando:

```
$ mvn clean package spring-boot:run
```

Ah! Para que você não se depare com uma tabela vazia, crie um arquivo SQL de nome `import.sql` no diretório `src/main/resources` com o seguinte conteúdo:

```
insert into Contato (id, nome, email) values (1, 'William Douglas', 'williamdouglas@algaworks.com');
insert into Contato (id, nome, email) values (2, 'Al Ries', 'alries@algaworks.com');
insert into Contato (id, nome, email) values (3, 'Mortimer J. Adler', 'mortimeradler@algaworks.com');
insert into Contato (id, nome, email) values (4, 'Christian Barbosa', 'christianbarbosa@algaworks.com');
```

Rodar a aplicação com dados de teste é bem melhor, não é mesmo? :)

## Benefícios do DevTools

DevTools é um módulo do Spring Boot que adiciona algumas ferramentas que são interessantes em tempo de desenvolvimento. No momento da criação do projeto, o DevTools foi um dos módulos que selecionei.

Um primeiro benefício dele é quanto a **configuração de propriedades úteis para desenvolvimento**.

Algumas bibliotecas suportadas pelo Spring Boot usam cache para melhorar sua performance. O Thymeleaf, por exemplo, faz o cache dos seus templates para não ter que recarregá-los a toda vez que forem requisitados. Isso, claro, é muito bom em produção, mas pode atrapalhar em tempo de desenvolvimento.

Para ajudar nessa situação o DevTools configura algumas propriedades com valores que são convenientes em tempo de desenvolvimento. Uma delas, por exemplo, é a que avisa ao Thymeleaf para não fazer cache dos seus templates. Ele deixa *spring.thymeleaf.cache* como false.

O segundo benefício que gostaria de comentar é a **reinicialização automática**. Cada arquivo criado nos diretórios que fazem parte do seu classpath serão monitorados e qualquer alteração neles acarretará em uma reinicialização do Spring Boot. Esses diretórios são, basicamente, os diretórios *src/main/java* e *src/main/resources*.

Lembrando que alterações em arquivos web estáticos e nos de templates não irão causar uma reinicialização. Eles não precisam.

Um último benefício que quero comentar é o fato do DevTools conter um servidor embarcado do **LiveReload**. O que esse servidor faz é enviar um aviso para o navegador dizendo que os nossos arquivos estáticos ou os de templates foram alterados. E com isso o navegador atualiza nossa página sozinho.

Para que isso funcione, além do servidor embarcado, é preciso que você instale uma extensão do LiveReload em seu navegador. Você pode encontrá-la no site [livereload.com](http://livereload.com/extensions/). Logo após a instalação da extensão, você deve acessar a aplicação e clicar no icone do LiveReload.

## Spring Boot VS Spring MVC

Quem aprende Spring começando com Spring Boot pode pensar que ele é um framework web. Isso é porque muitos exemplos com o Spring Boot, como o nosso caso, são de aplicações web.

Acontece que usar Spring Boot, sem o Spring MVC, não faz, do seu projeto, um projeto web.

Spring Boot e [Spring MVC](https://blog.algaworks.com/spring-mvc/) são frameworks diferentes. O primeiro nos ajuda com tarefas de infraestrutura do nosso projeto e o segundo nos ajuda a tratar requisições web.

## Como publicar sua aplicação feita com Spring Boot no Heroku

Para fechar com chave de outro, vamos publicar nossa aplicação na internet. Vou dar algumas dicas de como fazer isso utilizando o [Heroku](https://www.heroku.com/).

O Heroku é um serviço de hospedagem que já nos fornece toda a plataforma – como, por exemplo, questões de segurança e o próprio JDK instalado – que precisamos para publicar nosso projeto.

Caso você nunca tenha utilizado o Heroku, então vai precisar de três coisas. A primeira é fazer a instalação do [Git](https://git-scm.com/downloads) (gerenciador de código-fonte). A segunda é fazer o [cadastro no site](https://signup.heroku.com/). Por último, você vai precisar [instalar o Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) (para disponibilizar o comando *heroku* no seu terminal).

Depois, você pode rodar os seguintes comandos no seu terminal:

```
$ cd /pasta/do/projeto/gestao-contatos

$ git init; git add .; git commit -m "Versão concluída."

$ heroku login # login pelo terminal (utilize mesmo usuário e senha do cadastro)

$ heroku create # cria a sua aplicação lá no Heroku

$ git push heroku master # envia sua aplicação para o Heroku

$ heroku open # abre sua aplicação no navegador
```

Veja aplicação rodando no Heroku:

[![Aplicação com Spring Boot - Lista de contatos](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/Aplicacao-com-Spring-Boot-Lista-de-contatos.png)](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/Aplicacao-com-Spring-Boot-Lista-de-contatos.png)

## Conclusão

O Spring Boot traz grandes benefícios para quem trabalha com Spring e, nesse artigo, passei um pouco disso.

Vimos sobre o que é o Spring Boot, instalamos o STS e já partimos para o desenvolvimento de uma pequena aplicação para listagem de contatos. Depois, conversei com você sobre o DevTools, que são ferramentas que facilitam nosso trabalho em tempo de desenvolvimento.

Por último, ainda publicamos a aplicação na internet através do Heroku.

Com pouco código e nenhuma configuração, já foi possível publicar uma aplicação, com Spring Boot, na internet. Como você pôde comprovar, ele veio para nos deixar focados no que mais importa: as regras da nossa aplicação.

Se você gostou do que viu nesse artigo, então imagino que vai gostar muito também do nosso e-book **Produtividade no Desenvolvimento de Aplicações Web Com Spring Boot**:

[![Produtividade no desenvolvimento de aplicações web com Spring Boot.](https://algaworks-blog.s3.amazonaws.com/wp-content/uploads/FN013-CTA-Lead-Magnet-Img02.png)](http://cafe.algaworks.com/livro-spring-boot/)

Nele temos muito mais detalhes do Spring Boot.

Espero que tenha gostado do artigo. No mais, um abraço pra você e até uma próxima!

**PS**: você pode baixar o código-fonte de exemplo no nosso GitHub: [http://github.com/algaworks/artigo-spring-boot](https://github.com/algaworks/artigo-spring-boot).

[![img](https://secure.gravatar.com/avatar/98def809dd1ff0d88623373ddc725475?s=80&d=mm&r=g)](https://blog.algaworks.com/author/alexandre-afonso/)

### [Alexandre Afonso](https://blog.algaworks.com/author/alexandre-afonso/)

Trabalha como programador Java há mais de uma década, principalmente com desenvolvimento de sistemas corporativos. Além de colaborar com o blog, também é instrutor de vários cursos de Java na AlgaWorks.