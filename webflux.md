# Spring Webflux

Both web frameworks mirror the names of their source modules (spring-webmvc and spring-webflux) and co-exist side by side in the Spring Framework.
Reasons:
- The need for a non-blocking web stack to handle concurrency with a small number of threads and scale with fewer hardware resources.
- Functional programming.

Spring Webflux permite trabalhar com programação reativa em aplicações Java com Spring.

Spring MVC trabalha de forma síncrona e bloqueante.
Ou seja, se o cliente enviar mais duas requisições posteriores a primeira, essas ficarão bloqueadas até que a Requisição 1 seja finalizada.
Em uam aplicação assíncrona e não bloqueante, se o mesmo cliente do exemplo anterior enviar ao servidor a mesma Requisição 1, solicitando um grande volume de dados e em seguida, enviar mais duas Requisições, a Requisição 1 não mais bloqueará as requisições posteriores.

O Spring Webflux é baseado no projeto Reactor, que é uma biblioteca de programação reativa sem bloqueio para a JVM. Assim como o Reactor, ele possui dois tipos: Flux e Mono (publisher implementations). Lembrando que na programação reativa trabalha-se com fluxos e não com dados. Sendo assim, o tipo Flux consiste em um fluxo (stream) de 0 a N elementos e o tipo Mono consiste em um fluxo (stream) de 0 ou 1 elemento apenas.

a WebFlux API accepts a plain Publisher as input, adapts it to a Reactor type internally, uses that, and returns either a Flux or a Mono as output.

## O Manifesto Reativo
Foi publicado em 2014, é uma resposta à necessidade de utilizar melhor os recursos de hardware.
Esse manifesto prevê quatro elementos que definem um sistema reativo:

1. Responsivo: o sistema responde em tempo hábil se possível;

2. Resiliente: o sistema permanece responsivo quando ocorre uma falha;

3. Elástico: o sistema permanece responsivo sob carga variável;

4. Orientado a mensagens: sistemas reativos passam mensagens assíncronas para estabelecer os limites entre os componentes, o que garante baixo acoplamento, isolamento e localização transparente.

## Programming Models

The spring-web module contains the reactive foundation that underlies Spring WebFlux, including HTTP abstractions, Reactive Streams adapters for supported servers, codecs, and a core WebHandler API comparable to the Servlet API but with non-blocking contracts.

## Componentes Web Server-Side

Quando se utiliza o Spring Webflux, há duas maneiras de se criar componentes Web Server-Side: forma tradicional (anotações)e com estilo funcional (Serviço reativo funcional), Functional Endpoints: Lambda-based, lightweight, and functional programming model: onde os componentes são declarados via funções para rotear e manipular solicitações (Handler e Router). O Handler é onde manipula-se as requisições e defini-se os retornos como retorno HTTP, Content-Type e Body da resposta. Já o Router é o componente onde defini-se as rotas através de funções, recebendo como argumento do método o Handler já pré-definido para especificar o retorno para cada rota criada.

Annotated Controllers: One notable difference is that WebFlux also supports reactive @RequestBody arguments.

## Performance

Reactive and non-blocking generally do not make applications run faster. On the whole, it requires more work to do things the non-blocking way and that can slightly increase the required processing time.
The key expected benefit of reactive and non-blocking is the ability to scale with a small, fixed number of threads and less memory.

## Servers

Spring WebFlux is supported on Tomcat, Jetty, Servlet 3.1+ containers, as well as on non-Servlet runtimes such as Netty and Undertow.

## “Reactive”

The term, “reactive,” refers to programming models that are built around reacting to change — network components reacting to I/O events, UI controllers reacting to mouse events, etc. In that sense, non-blocking is reactive, because, instead of being blocked, we are now in the mode of reacting to notifications as operations complete or data becomes available.

A Programação Reativa é um paradigma funcional, orientado a eventos, não bloqueante, assíncrono e centrado no conceito de processamento de data streams (fluxos de dados) e é um estilo de programação cujas bases foram estabelecidas pelo Manifesto Reativo.

## Reactive Web Client
WebClient, introduced in Spring 5, is a non-blocking client with support for reactive streams.
public class EmployeeWebClient {
    WebClient client = WebClient.create("http://localhost:8080");
    // ...
}

Retrieving a Single Resource:

Mono<Employee> employeeMono = client.get()
  .uri("/employees/{id}", "1")
  .retrieve()
  .bodyToMono(Employee.class);

employeeMono.subscribe(System.out::println);

## Spring WebFlux Security

@EnableWebFluxSecurity
public class EmployeeWebSecurityConfig {
    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(
      ServerHttpSecurity http) {
        http.csrf().disable()
          .authorizeExchange()
          .pathMatchers(HttpMethod.POST, "/employees/update").hasRole("ADMIN") // restrict this endpoint to ADMIN role users
          .pathMatchers("/**").permitAll()
          .and()
          .httpBasic();
        return http.build();
    }
}

---

Segue tutorial:

# https://www.youtube.com/watch?v=jW1YdAb3GZo&list=PL8iIphQOyG-CyD9uuRTMiqxEut5QAKHga

Permite programar de forma reativa dentro do Spring!

Spring MVC - sincrona e bloqueante - abre uma thread por requisição e chega a um limite que cria uma fila de espera.

Assincrona - não bloqueante - requisições processadas parelelamente!
Biblioteca Reactor 

Tipos retornáveis:
	Mono: buscar um registro
	Flux: Buscar mais de um dado (tabela)

O webflux pode ser utilizado em conjunto com o spring mvc;

Servlet container
Spring mvc = Tomcat
Webflux = Netty -> servidor que trabalha com tempo de execução assincrona.

## API REST com Spring Webflux e MongoDB: parte1 - iniciando o projeto

É necessário a Dependencia: Reactive Web
spring-boot-starter-webflux

É preciso definir a conexão com o MongoDB no properties:
spring.data.mongodb.uri=???
(conexao com o banco)

## API REST com Spring Webflux e MongoDB: parte2 - criando um Document e Repository

Exemplo de pacotes dentro do projeto: document, repository, services

Na entidade:
@Document (org.springframework.data.mongodb.core.mapping.Document)
Para fazer o mapeamento do dado do banco com o objeto!

No repository:
interface ... extends ReactiveMongoRepository<Entidade, String>
Já disponibiliza métodos prontos para alterar dados do banco!

@Component
Classe anotada com isso é um bean gerenciado pelo Spring.

## API REST com Spring Webflux e MongoDB: parte3- criando um service

O service inicia com uma interface com métodos como findAll(), findById(), save().
Mas na programação assíncrona (webflux) não se retorna
List<T>
mas sim
Flux<T>			(reactor.core.publisher.Flux)
ou
Mono<T>			(reactor.core.publisher.Mono)

Já a classe que implementa essa interface Service precisa ser anotada com
@Service
E conter o repositório, logo, precisa de uma ponto de injeção para a interface Repository, usamos a anotação:
@Autowired

## API REST com Spring Webflux e MongoDB: parte4- criando o controller
Vamos construir o endpoint, criando o Controller

A classe Controller precisa ser anotada com
@RestController
para responder as requisições. Para cada método da requisição anotamos com 
@GetMapping() - essa anotação depende de parâmetros como "value".

No Controller temos que criar um ponto de injeção para adicionar a interface Service (@Autowired).
Os métodos do Controller também retornam
Flux<T>
que é retornado pelo Service.

Depois disso, os acessos as urls definidas no "value" pelo navegador devem funcionar.

## API REST com Spring Webflux e MongoDB: parte5- criando um serviço reativo funcional

Criando endpoints com um Estilo funcional de programação no webflux
- router
Rotear as solicitações
- handler
Quais são os métodos que controlam as regras de negócio

Handler:
A ideia é substituir a class anotada com o RestController pelo Handler.

 
@Component -> bean gerenciado pelo spring
class ...Handler {

Essa classe possui uma instância de Service (@Autowired)

Classe ServerResponse: faz parte do spring reativo.
org.springframework.web.reactive.function.server.ServerResponse
Representa uma resposta http do servidor.

public Mono<ServerResponse> findAll(ServerRequest request)
return ok().contentType(MediaType.APPLICATION_JSON)
.body(service.findAll(), Classe.class);

Ainda é necessário utilizar o Mono.

Em métodos que recebem os objetos a classe ServerRequest tem o método .bodyToMono(Classe.class) para atribuir o objeto a uma variável Mono<Class>.

public Mono<ServerResponse> save(ServerRequest request){
	final Mono<Playlist> playlist = request.bodyToMono(Playlist.class);
	return ok()
	.contentType(MediaType.APPLICATION_JSON)
	.body(fromPublisher( playlist.flatMap( service::save), Playlist.class));
}

O método .flatMap() transforma o objeto emitido por este Mono assincronamente.

Router:
@Configuration  -> é uma classe de configuração do spring.
class ...Router {

@Bean
	public RouterFunction<ServerResponse> route(PlaylistHandler handler){
		return RouterFunctions
				.route(GET("/playlist").and(accept(MediaType.APPLICATION_JSON)), handler::findAll)
				.andRoute(GET("/playlist/{id}").and(accept(MediaType.APPLICATION_JSON)), handler::findById)
				.andRoute(POST("/playlist").and(accept(MediaType.APPLICATION_JSON)), handler::save);
			
	}
Os endpoints definidos aqui conflitam com os endpoints do RestController (comentar lá).

## API REST com Spring Webflux e MongoDB: parte6- Events Stream

Stream de eventos = Events stream

Para isso vamos utilizar o RestController

Fluxo reativo!

@GetMapping(value="/playlist/webflux", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
	public Flux<Tuple2<Long, Playlist>> getPlaylistByWebflux(){
	System.out.println("---Start get Playlists by WEBFLUX--- " + LocalDateTime.now());
	Flux<Long> interval = Flux.interval(Duration.ofSeconds(10));
	Flux<Playlist> playlistFlux = service.findAll();

	return Flux.zip(interval, playlistFlux);
}

GetMapping: define a uri
produces: para definir o tipo stream de eventos
interva: um fluxo do tipo long, intervalo de cada stream que vamos enviar para o usuário.
Flux.zip: permite retornar para o usuário mais de uma variável, usado em conjunto com a classe Tuple2.

Esse método vai enviar ao usuário de 1 em 1 em intervalos de 10 segundos, todos os registros retornados do findAll().

Mas a principal questão aqui é a opção assincrona que faz com que todas as requisições sejam respondidas, e nenhuma requisição fique presa por causa da sincronidade do servidor;

---

Fontes: 

https://medium.com/@michellibrito/spring-webflux-f611c8256c53
https://www.youtube.com/watch?v=jW1YdAb3GZo&list=PL8iIphQOyG-CyD9uuRTMiqxEut5QAKHga
https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html
https://www.baeldung.com/spring-webflux
https://atitudereflexiva.wordpress.com/2020/12/08/spring-boot-introducao-a-programacao-reativa-com-webflux/
