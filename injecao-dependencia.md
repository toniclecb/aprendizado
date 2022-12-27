# Injeção de Dependências com Spring: o guia essencial que você precisa saber

AlgaWorks - https://www.youtube.com/watch?v=mzMgLD3YrfE


Padrão Dependency Injection e Principio da inversão de dependencia
são parecidos mas não são a mesma coisa

**Dependencia**
quando uma classe depende (usa) de outra classe

Existe um problema de boiler plate quando precisa utilizar uma classe.
Exemplo, quando instanciamos uma classe dentro de outra ferimos o principio do aberto e fechado (SOLID).
Também fere o padrão de inversão de dependência (classes devem depender de classes de auto nível).
Também existe uma dificuldade de fazer testes unitários deste modo.

**Inversão de controle - IOC**

A instanciação de classes não deve ficar dentro de classes.
No Spring, ao invés de usar "new ApiService()" internamente nos métodos, faz o seguinte: Adiciona um novo campo na classe (campo ApiService), e cria-se um construtor que recebe este tipo e atribui ao novo campo da classe.
Fazendo isso a gente invertou o controle.
A classe de regra de negócio não precisa conter esse controle.

Isso acima ainda fere o Open-Closed, vamos usar Dependency Invertion DIP
o objetivo é desaclopar classes do nosso projeto!
Basicamente vamos criar uma Interface (abstração)
Todos os "services" do sistema devem implementar uma abstração!
Logo, alterações nas implementações das abstrações não vão interferir nos "services".
Exemplo, caso se crie uma nova implementação (um novo gateway de pagamento foi integrado) basta criar apenas a implementação.

No spring
Bean: objeto gerenciado pelo Spring
A anotação @Configuration indica que a classe define os beans com métodos:
Exemplo:
@Bean
public GatewayService gatewayService(){
	return PagSeguroService("qbn12jk352jk3");
}
Toda vez que o Spring identificar que precisa de uma instância de GatewayService ele "pegará" deste método!
O Spring automatiza alguns processos quanto as instâncias, exemplo:
@Bean
public VendaService vendaService(GatewayService gatewayService){
	return new VendaService(gatewayService);
}
Aqui o Spring com seu container de IOC vai pegar os beans gerenciados e disponibilizar para os métodos que precisam, como este acima!

O método acima que define VendaService como um Bean pode ser eliminado, e ao invés disso, a classe VendaService pode ser anotada com @Component, assim ela também vira um bean gerenciado pelo Spring, e a injeção que ocorre dentro dela pelo construtor (da classe GatewayService) também funcionará!

Você pode adquirir um objeto bean que é gerenciado pelo Spring, pela variável "applicationContext" returnada do método SpringApplication.run() (inicialização de uma aplicação Spring, dentro da classe anotada com @SpringBootApplication)
VendaService vs = applicationContext.getBean(VendaService.class);

Não faz sentido deixar o Spring gerenciar beans de entidades, essas devem ser gerenciadas pelos "services"!

**Anotação @Primary**
Utilizada junto com @Bean, ela serve como decisora no momento do Spring escolher qual classe deve ser injetada, isso se o bean se repete (tem duas configs. para a mesma abstração).
Existem outras formas de fazer essas escolhas.

Por que colocar a dependencia via construtor?
Obriga a classe ser instanciada com a dependencia, e ela não vai existir sem.

**Tipos diferentes de injeção**

Método
Basta colocar a classe como parâmetro do método e anotar o método com @Autowired 

Propriedade da classe
Na declaração do campo também podemos anotar com @Autowired e o Spring fará a injeção!	
