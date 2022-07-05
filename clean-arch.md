1--- https://medium.com/luizalabs/descomplicando-a-clean-architecture-cf4dfc4a1ac6

A. Clean Architecture

Objetivo da Clean Architecture é fornecer uma maneira de organizar o código de forma que encapsule a lógica de negócios, mas mantenha-o separado do mecanismo de entrega.

As vantagens de utilizar uma arquitetura em camadas são muitas, porém podemos pontuar algumas:

Testável. As regras de negócios podem ser testadas sem a interface do usuário, banco de dados, servidor ou qualquer outro elemento externo.
Independente da interface do usuário. A interface do usuário pode mudar facilmente, sem alterar o restante do sistema. Uma UI da Web pode ser substituída por uma UI do console, por exemplo, sem alterar as regras de negócios.
Independente de banco de dados. Você pode trocar o Oracle ou SQL Server, por Mongo, BigTable, CouchDB ou qualquer outro. Suas regras de negócios não estão vinculadas ao banco de dados.
Independente de qualquer agente externo. Na verdade, suas regras de negócios simplesmente não sabem nada sobre o mundo exterior, não estão ligadas a nenhum Framework.

A separação de camadas poupará o desenvolvedor de muitos problemas futuros com a manutenção do software, a regra de dependência bem aplicada deixará seu sistema completamente testável. Quando um framework, um banco de dados, ou uma API se tornar obsoleta a substituição de uma camada não será uma dor de cabeça, além de garantir a integridade do core do projeto.

2--- https://medium.com/@renicius.pagotto/clean-architecture-e-suas-premissas-6beb933c72b1

B. - Conceitos que devem ser conhecidos
Acoplamento
Quando as diferentes partes que compõe um software possuem um alto grau de dependência, dizemos então que possui um alto acoplamento. Isso afeta a manutenção e testabilidade do sistema. Outro ponto é a dificuldade de realizar mudanças ou ajustes no comportamento do sistema.

Coesão
Termo designado para as responsabilidade de cada parte do sistema. Quando uma parte do nosso sistema, realiza diferentes tarefas, conceitualmente, possui uma baixa coesão.

3---
C. - Camadas da clean architecture
Entidade/Entities: Lugar onde ficam os objetos de domínio, regras de negócio, contratos de serviços e outras funções referentes a manipulação de dados do sistema. Aqui poderiam residir as bibliotecas que são consumidas pela aplicação, mas que a aplicação não irá modificar.

Casos de uso/Use cases: Nessa camada estão os objetos que representam uma ação dentro do sistema, sendo chamados de casos de uso. Esses casos de uso realizam a orquestração de fluxo de dados, ou seja, direcionam os dados para aplicação da regra de negócio, validações e posteriormente para as persistências.

Adaptadores de interface/Interface Adapters: Camada responsável, por realizar a comunicação entre as entidades e os componentes externos da aplicação, como por exemplo, a interface do usuário (UI). Um exemplo seria o uso do AutoMapper, onde eu poderia controlar as estruturas transmitidas entre User Cases e Entities com o interface do usuário, por exemplo.

Frameworks e drivers/ Frameworks & Drivers: Camada composta de ferramentas, como por exemplo, o banco de dados. Nessa camada, a ideia é ter o mínimo de código possível, apenas o necessário para interligar as camadas e injetar as implementações necessárias nas camadas interiores.

4--- https://imasters.com.br/back-end/introducao-clean-architecture

D. Dependências
image: https://static.imasters.com.br/wp-content/uploads/2018/10/07222901/conn.jpg
Na imagem acima, temos o desenho de como a Clean Architecture funciona. Note que as setas horizontais da imagem que representam as dependências entre as camadas vêm de “fora para dentro”, ou seja, a camada Framework “enxerga” somente a Interface Adapters, que por sua vez “enxerga” somente a User Cases, que finalmente “enxerga” apenas a Entites. Esta é a principal regra da Clean Architecture, e talvez sua única: o Princípio da Dependência.

As camadas internas não devem ter qualquer dependência das externas, nem indiretas, como nomes de variáveis, funções ou termos.

Provavelmente você terá mais abstrações nas camadas internas e mais implementações nas externas, utilizando injeção de dependência para fazer tudo funcionar.

5--- https://balta.io/blog/clean-code

E. Definições básicas, não são conceitos da Clean, mas são conceitos importantes de um bom Desenvolvedor

Siga as convenções
Se você começou agora em um projeto ou acabaram de definir suas convenções, siga-as! Se utilizam por exemplo constantes em maiúsculo, enumeradores com E como prefixo, não importa! Siga sempre os padrões do projeto.

KISS
Mantenha as coisas simples! Este conceito vem até de outro livro, e particularmente acho que é a base de uma boa solução. Normalmente tendemos a complicar as coisas que poderiam ser muito mais simples.

Então, Keep It Stupid Simple (Mantenha isto estupidamente simples - KISS)!

Regra do escoteiro
"Deixe sempre o acampamento mais limpo do que você encontrou!" O mesmo vale para nosso código. Devolva o código melhor do que você o obteve. Se todo desenvolvedor no time tiver esta visão, e devolver um pedacinho de código melhor do que estava antes, em pouco temos teremos uma grande mudança.

Causa raiz
Sempre procure a causa raiz do problema, nunca resolva as coisas superficialmente. No dia-a-dia, na correria, tendemos a corrigir os problemas superficialmente e não adentrar neles, o que muitas vezes causa o re-trabalho!

6---

F. Regras de design
Mantenha dados de configuração em alto nível
Algo que toda aplicação tem são suas configurações, como as conhecidas ConnectionStrings. Tente sempre deixar estas configurações ou o parse delas em um nível mais alto possível.

Evite sobrescrever (ou alterar) configurações em métodos dentro de Controllers ou algo do tipo. Se possível, mantenha esta passagem no método principal, no início da aplicação e não mexa mais nisto!
Se existe uma configuração com o nome de um banco, por exemplo, não crie variáveis internas a Controllers ou Persistences com outro nome de banco, se existe mais de uma variável coloque tudo no arquivo de alto nível de configuração.


7---

G. Polimorfismo no lugar de IFs
Um IF ou condicional, como o nome diz, traz uma tomada de decisão a nossa aplicação, o que implica no aumento da complexidade da mesma. No geral devemos evitar o uso excessivo destes.
public class Pagamento {
    public bool PodeSerPago() {
        if(tipo == ETipoPagamento.Boleto)
        {
            if(vencimento.Day != IsWeekend())
                return true;
Caso hajam mais formas de pagamento futuramente, como tratariamos este código? Encheriamos de IF?
Uma forma de melhorar isso:
public class PagamentoBoleto : Pagamento {
    public override bool PodeSerPago() {
        if(vencimento.Day != IsWeekend())
            return true;
    }
}
A solução mais plausível é derivar da classe base Pagamento criando o PagamentoBoleto que sobrescreve o método PodeSerPago, dando uma nova funcionalidade a ele.

8--- https://engsoftmoderna.info/artigos/arquitetura-limpa.html

H. Invertendo o Fluxo de Controle

Em uma Arquitetura Limpa, fluxos de controle de fora para dentro são implementados de forma natural, pois eles seguem o mesmo sentido da Regra de Dependência. Por exemplo, uma camada mais externa Y pode criar um objeto de um tipo mais interno X e então chamar um método desse objeto.
No entanto, em alguns cenários, um caso de uso pode ter que chamar um método de uma classe de uma camada mais externa. Para ficar claro, suponha que um caso de uso precise enviar um mail. Antes de mais nada, vamos supor que existe no sistema uma classe, de uma camada mais externa, chamada MailServiceImpl e com um método send:
public class MailServiceImpl {
    public void send(String msg);
}
No entanto, veja que esse exemplo implica em um fluxo de fora para dentro: o caso de uso tem que declarar uma variável de uma classe de uma camada mais externa, o que contraria a regra da dependência!
A solução implica em ter uma interface na camada de caso de uso chamada MailServiceInterface com um método send(String msg).
package CasosDeUso;

public interface MailServiceInterface {
    void send(String msg);
}

Essa interface foi criada para funcionar como uma abstração para o serviço de envio de mail. Ou seja, para evitar que o caso de uso tenha que se acoplar a uma classe concreta desse serviço.
Além disso, como MailServiceInterface pertence à camada Caso de Uso, as outras classes dessa camada podem chamar send sem violar a Regra de Dependência.
Prosseguindo, a classe MailServiceImpl deve implementar a interface MailServiceInterface.
import CasosDeUso.MailServiceInterface;

public class MailServiceImpl implements MailServiceInterface {
    public void send(String msg);
}
Essa implementação não viola a Regra de Dependência, pois uma classe de uma camada mais externa (MailServiceImpl) está dependendo de um elemento de código de uma camada mais interna. No caso, esse elemento é uma interface (MailServiceInterface).

9--- https://www.eduardopires.net.br/2013/04/orientacao-a-objeto-solid/

I. – S.O.L.I.D.
SOLID é um acrônimo dos cinco primeiros princípios da programação orientada a objetos e design de código identificados por Robert C. Martin (ou Uncle Bob) por volta do ano 2000.

– SRP: Single Responsibility Principle - Uma classe deve ter um, e somente um, motivo para mudar.
– OCP: Open/Closed Principle - Você deve ser capaz de estender um comportamento de uma classe, sem modificá-lo.
– LSP: Liskov Substitution Principle - As classes base devem ser substituíveis por suas classes derivadas.
– ISP: Interface Segregation Principle - Muitas interfaces específicas são melhores do que uma interface única.
– DIP: Dependency Inversion Principle - Dependa de uma abstração e não de uma implementação.

Os princípios SOLID devem ser aplicados para se obter os benefícios da orientação a objetos, tais como:

Seja fácil de se manter, adaptar e se ajustar às alterações de escopo;
Seja testável e de fácil entendimento;
Seja extensível para alterações com o menor esforço necessário;
Que forneça o máximo de reaproveitamento;
Que permaneça o máximo de tempo possível em utilização.
Utilizando os princípios SOLID é possível evitar problemas muito comuns:

Dificuldade na testabilidade / criação de testes de unidade;
Código macarrônico, sem estrutura ou padrão;
Dificuldades de isolar funcionalidades;
Duplicação de código, uma alteração precisa ser feita em N pontos;
Fragilidade, o código quebra facilmente em vários pontos após alguma mudança.


10--- https://medium.com/desenvolvendo-com-paixao/o-que-%C3%A9-solid-o-guia-completo-para-voc%C3%AA-entender-os-5-princ%C3%ADpios-da-poo-2b937b3fc530

J. SRP — Single Responsibility Principle

Esse princípio declara que uma classe deve ser especializada em um único assunto e possuir apenas uma responsabilidade dentro do software, ou seja, a classe deve ter uma única tarefa ou ação para executar.
Classe Deus: Na programação orientada a objetos, é uma classe que sabe demais ou faz demais.

A violação do Single Responsibility Principle pode gerar alguns problemas, sendo eles:
- Falta de coesão — uma classe não deve assumir responsabilidades que não são suas;
- Alto acoplamento — Mais responsabilidades geram um maior nível de dependências, deixando o sistema engessado e frágil para alterações;
- Dificuldades na implementação de testes automatizados — É difícil de “mockar” esse tipo de classe;
- Dificuldades para reaproveitar o código;

O princípio da responsabilidade única não se limita somente a classes, ele também pode ser aplicado em métodos e funções, ou seja, tudo que é responsável por executar uma ação, deve ser responsável por apenas aquilo que se propõe a fazer.

11---

K. OCP — Open-Closed Principle

Princípio Aberto-Fechado — Objetos ou entidades devem estar abertos para extensão, mas fechados para modificação, ou seja, quando novos comportamentos e recursos precisam ser adicionados no software, devemos estender e não alterar o código fonte original.

Não seria mais fácil apenas acrescentar mais um IF e verificar o novo tipo de funcionário PJ aplicando as respectivas regras? Sim, e provavelmente essa seria a solução que programadores menos experientes iriam fazer. Mas, esse é exatamente o problema! Alterar uma classe já existente para adicionar um novo comportamento, corremos um sério risco de introduzir bugs em algo que já estava funcionando.

Open-Closed Principle também é base para o padrão de projeto Strategy, a sua principal vantagem é a facilidade na adição de novos requisitos, diminuindo as chances de introduzir novos bugs — ou bugs de menor expressão — pois o novo comportamento fica isolado, e o que estava funcionando provavelmente continuara funcionando.

12---

L. LSP— Liskov Substitution Principle

Princípio da substituição de Liskov — Uma classe derivada deve ser substituível por sua classe base.

Para facilitar o entendimento, veja na prática: passando como parâmetro tanto a classe pai como a classe derivada e o código continua funcionando da forma esperada (quando criado um método com List<> ao invés de ArrayList<>).

Exemplos de violação do LSP:
Sobrescrever/implementar um método que não faz nada;
Lançar uma exceção inesperada;
Retornar valores de tipos diferentes da classe base;

Para não violar o Liskov Substitution Principle, além de estruturar muito bem as suas abstrações, em alguns casos, você precisara usar a injeção de dependência e também usar outros princípios do SOLID, como por exemplo, o Open-Closed Principle e o Interface Segregation Principle.

Seguir o LSP nos permite usar o polimorfismo com mais confiança. Podemos chamar nossas classes derivadas referindo-se à sua classe base sem preocupações com resultados inesperados.

13---

M. ISP — Interface Segregation Principle
Princípio da Segregação da Interface — Uma classe não deve ser forçada a implementar interfaces e métodos que não irão utilizar.

Esse princípio basicamente diz que é melhor criar interfaces mais específicas ao invés de termos uma única interface genérica.
O entendimento comum é que uma classe deve manter a sua interface simples. Deve compô-la apenas com os métodos necessários para garantir seu tipo e que estejam relacionados com sua responsabilidade.

Exemplo prático: a interface ITelefone tem o método tirarFoto(), e as classes SmartPhone e TelefoneSimples implementam essa interface, mas o TelefoneSimples não sabe o que fazer com esse método (poderia lançar um NotImplementedException). Para resolver basta criar interfaces específicas para as classes (ITelefoneComum e ITelefoneCelular).

No entanto, essa dependência pode ser externa. Você pode estar escrevendo um componente para acessar um banco de dados, ou a um sistema de terceiros, por exemplo. Em casos como esse, não há muito o que fazer. Você não tem controle das dependências e nem tem como diminuí-las na fonte. Neste caso, convém adotar outros padrões de projeto a ponto de deixar esse acoplamento o mais fraco possível e invisível para quem consome.

14---

N. DIP — Dependency Inversion Principle
Princípio da Inversão de Dependência — Dependa de abstrações e não de implementações.

De acordo com a definição do DIP, um módulo de alto nível não deve depender de módulos de baixo nível, ambos devem depender da abstração. Então, a primeira coisa que precisamos fazer é identificar no nosso código qual é o módulo de alto nível e qual é o módulo de baixo nível. Módulo de alto nível é um módulo que depende de outros módulos.

1. Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender da abstração.
2. Abstrações não devem depender de detalhes. Detalhes devem depender de abstrações.

Importante: Inversão de Dependência não é igual a Injeção de Dependência, fique ciente disso! A Inversão de Dependência é um princípio (Conceito) e a Injeção de Dependência é um padrão de projeto (Design Pattern).

Exemplo:
Existe uma classe UsuarioPersistence que faz um MySQLConnection() no seu construtor;
Temos um alto nível de acoplamento, isso acontece porque a classe tem a responsabilidade de criar uma instância da classe MySQLConnection! Se quiséssemos reaproveitar essa classe em outro sistema, teriamos obrigatoriamente de levar a classe MySQLConnection junto.
Para resolver isso refatoramos a nossa classe e em vez de instanciar a gente recebe MySQLConnection via parâmetro no construtor!
Isso é chamado de Injeção de Dependência!
Apesar de termos usado a injeção de dependência para melhorar o nível de acoplamento do nosso código, ele ainda viola o princípio da inversão de dependência! — lembre-se, um não é igual ao outro.
Além de violar o DIP, se você prestar atenção na forma que o exemplo foi codificado irá perceber que ele também viola o Open-Closed Principle.
Por que nosso exemplo refatorado viola o Dependency Inversion Principle?
Porque estamos dependendo de uma implementação e não de uma abstração, simples assim.
Como refatoramos nosso exemplo para utilizar o DIP?
Criando uma interface DBConnectionInterface com os métodos que a classe UsuarioPersistence precisa utilizar da MySQLConnection.
A interface vai permitir a implementação da classe MySQLConnection e de outras (OracleConnection, PostGresConnection, etc.)
Para resolver de vez, refatoramos a nossa classe e em vez de instanciar a gente recebe DBConnectionInterface via parâmetro no construtor!
Agora, UsuarioPersistence não tem a mínima ideia de qual banco de dados a aplicação irá utilizar.