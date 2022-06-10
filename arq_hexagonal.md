# Introdução a Arquitetura Hexagonal

Fonte: https://www.udemy.com/course/introducao-a-arquitetura-hexagonal/learn/lecture/15421460

Elaborado e documentado em 2005 por Alistair Cockburn:
● um dos autores do manifesto agil 2001.
● autor da metodologia Cristal Clear.

Nome conhecido: "Ports and Adapters Patterns"

## Teoria e Fundamentos

Padrão de projeto -> solução geral reutilizável para um problema que ocorre com frequência;
Padrão Arquitetural! -> regras e princípios que definem como será a organização estrutural abstrata dos componentes significativos de um software, seus relacionamentos e suas interfaces externas.

"Guideline" -> simplificar processos específicos de acordo rotinas definidas ou práticas.

**Para que serve?**

Projetar e construir aplicativos de software, estabelecendo uma arquitetura moderna, robusta e altamente flexível, orientadas pelas premissas básicas da filosofia de desenvolvimento ágil: 
1. desenvolvimento orientado a TDD;
2. foco nos requisitos de negócio;
3. adiar decisões técnicas o máximo possível;

**Qual objetivo?**

Criar uma soluções de software que tenha uma arquitetura que permita igualmente ser executada por usuários, programas, testes automatizados, e que seja desenvolvido e testado isoladamente de seus dispositivos externos de execução eventuais, fazendo que com a equipe de desenvolvimento foque no desenvolvimento do requisitos de negócio, ignorando dependências externas técnicas e infra estruturais.

## Isolamento

**Separation of concerns - SoC**
É um princípio básico da engenharia de software, que visa separar preocupações, ou seja, modularizar uma solução de forma que cada módulo esteja focado em resolver apenas um único problema. Separar preocupações é imprescindível para quem deseja elaborar uma arquitetura flexível, manutenível e sustentável.

**Hexágono**

Divisão em três partes:
1. Lado superior esquerdo
2. Centro hexágono
3. Lado inferior direito

Podem ser considerados 3 módulos.

### Centro Hexágono
Temos as coisas que são importantes para o problema de negócio que a solução está tentando resolver.

É a parte mais importante do sistema.
Código que relaciona com lógica de negócios do contexto da solução.
Contéra código de oop/aop/fp que abstrai as regras de negócio do mundo real em modelo de objetos. Esta é a parte que queremos isolar de todas as outras partes (lados esquerdo e direito).

Deve ser totalmente agnóstico, independente de tecnologias, bibliotecas, interfaces gráficas, etc.
Mas pode ter dependências de frameworks de serviços gerais: logs, IoC, etc.

### Lado superior esquerdo

Toda requisição realizada deve ficar neste lado.
Lado intercambiável e flexível através do qual um ator externo irá interagir com a solução. Conterá código de tecnologia específica que irá disparar eventos, normalmente um pessoa usando uma GUI ou programa externo.

### Lado direito inferior

Lado intercambiável e flexível que fornecerá os serviços de infraestrutura. Código de tecnologia especifica, detalhes infraestruturais, normalmente código que interage com o banco de dados, faz chamadas para o sistema de arquivos, chamadas HTTP e outros aplicativos.

## Atores

Fora do hexágono, temos qualquer coisa do mundo real com a qual o aplicativo interage.

**1. Ator Primário Condutor (Driver - Condutores)**
- Lado esquerdo superior;
- É aquele interage com o aplicativo para atingir um objetivo;
- Exemplos: Suites de testes, front-end desktop, front-end web, end-point rest e etc...

**2. Ator Secundário Conduzido (Driven - Conduzido)**
- Lado direito inferior;
- A interação é acionada pelo hexágono. 
- Um ator secundário fornece funcionalidades necessárias ao hexágono para processar a lógica de negócios. 
- Exemplos: banco de dados relacionais, nosql, serviços web http, stmp, sistema de arquivos e etc.

**2.1 Repositório**
- É aquele com o objetivo de enviar e receber informações. Por exemplo, um banco de dados.

**2.2 Destinatário**
- É aquele com o objetivo de somente enviar informações e esquecer. Por exemplo, um envio de email SMTP.

**Como descobrir novos atores?**
- “quem” aciona a conversa?
  - Se a resposta for "o ator", então é um condutor primário.
  - Se a resposta for "o aplicativo", o ator é um ator conduzido secundário.


## Dependências

- princípio de dependências: "somente de fora para dentro!"

Sendo assim, o lado esquerdo e o direito se torna totalmente **flexível, intercambiável** e dependente do centro.

1. Lado esquerdo, os atores primários dependem do hexágono.
2. Lado direito, os atores secundários dependem do hexágono.
3. O centro, o hexágono não depende de ninguém, só dele mesmo.

![image](https://user-images.githubusercontent.com/1041102/172965794-7426698b-25de-4356-921f-70fdeed2ed8b.png)


## Portas

A comunicação **"de fora para dentro"** deve ser feito única e exclusivamente através de uma metáfora chamada "porta".
as interações entre todos os **atores** e o **hexágono** são organizadas no **limite do hexágono (Edge)** pelo motivo de estarem interagindo com o aplicativo. Cada grupo de interações com um determinado objetivo / **intenção é uma porta**.

![image](https://user-images.githubusercontent.com/1041102/172966426-232c793d-379c-4a42-bd52-5d462a687e3d.png)

As portas são interface OOP, que polimorficamente definem um contrato de uso, e ao mesmo tempo isolam e encapsulam como aquele determinado serviço está sendo executado. 
Portanto, a arquitetura deve seguir as regras de encapsulamento polimórfico. 

Arquitetura hexagonal estabelece 2 tipos de portas:
1. Porta Primária Condutor (Driver)
2. Porta Secundária Dirigida (Driven)

### Porta Primária Condutor (Driver)

Chamados pelos atores primários que formam o lado do usuário da solução. Portas primárias formam a API.
São **funções e ou eventos** ofertados pela solução que permitem alterar objetos, ou atributos e execução de processos relacionado a lógica principal. 
Exemplos: adicionar item carrinho de compra, pagar fatura, transferência bancária e etc.

São nomeadas como **caso de usos** (Use Case).
Alistair Cockburn propôs que as portas condutores sejam mapeadas em formatos de caso de usos, uma vez que eles refletem exatamente as funções e ou eventos que solução precisa fazer.

### Porta Secundária Dirigida (Driven)

São chamados pelo hexágono para executar serviços externos.
Portas secundárias formam a SPI (Service Provider Interface) que é requerida pela solução. 
Possibilitam acessar dados em banco de dados relacionais, nosql, serviços web http, stmp, sistema de arquivos, por 
exemplos inserir cliente, pesquisar fatura, procurar cliente inativos e etc.

### Resumo

Todas as classes OOP que implementam a interfaces de **portas primárias** (casos de uso) devem estar **dentro do hexágono**, pois elas são agnósticos a tecnologias e são usadas para definir e redirecionar as chamadas externas para dentro das operações de negócio oferecidas pela solução.

Todas as classes OOP que implementam a interfaces de portas secundárias devem estar fora do hexágono, pois elas usam alguma tecnologia específica e convertem chamada de negócio, agnóstico a tecnologia em alguma necessidade infraestrutural e externa a solução.

**Hexagono 100% Isolado**
Assim, **essas interfaces atuam como isoladores explícitos** entre o interior e o exterior da solução, tanto para o lado esquerdo de entrada, quanto o lado direito de saída, cumprindo o principal objetivo da visão hexagonal que é ter “core” da solução 100% isolado, independente de tecnologia de GUI, dispositivos externos, serviços infraestruturais e podendo ser orientado a TDD.

## Adaptadores

Um adaptador é um componente de software que permite uma tecnologia externa interaja com uma porta do hexágono.

1. Adaptador Condutor (Driver)
2. Adaptador Dirigido (Driven)

### Adaptador Condutor (Driver)

É o componente usado para converter uma solicitação de tecnologia específica em uma solicitação agnóstica e pura de sistema para uma porta condutora, traduzindo dados de entradas externos para dentro da solução. 
**Objetivo**: Responsável por fazer integração do lado de fora para dentro do hexágono.

São classes OOP que usam frameworks, convertendo dados dessas filosofías externas para dentro do hexágono, repassando as operações para a porta primária. Exemplos: 
● classes teste junit
● classes desktop Swing ou JavaFx
● classes web JSF managed beans
● classes json end point rest jax-rs
● classes json end point rest spring mvc

Para cada porta condutora, deve haver pelo menos dois adaptadores: 
1. um para testar o comportamento via TDD.
2. um usando a tecnologia real requerida pela solução.

### Adaptador Dirigido (Driven)

É o componente usado para converter chamadas de dentro do solução para fora, usando serviços de infraestrutura tecnológicos externos a solução.
**Objetivo**: Responsável por fazer integração de dentro do hexágono para o lado de fora.

Os adaptadores dirigidos são classes OOP que implementam as interfaces de portas dirigidos e que usam frameworks e tecnologias específicas, dando o suporte para aquelas necessidades de chamadas externas. Exemplos: 
● Classe DAO via JDBC.
● Classe EAO via JPA.
● Classe envio via JavaMail usando SMTP.
● Classe envio de sms consumidor de um web services jax-ws.
● Classes cliente consumidor de um rest end point via jax-rs.
● Classes cliente consumidor de um rest end point via Sprint RestTemplate.

Para cada porta dirigidos devemos escrever pelo menos dois adaptadores: 
1. um para o dispositivo do mundo real.
2. outro para um simulado que imita o comportamento real, chamado de mock.

## Adaptadores simulados - Mock

Arquitetural hexagonal promove que a solução seja desenvolvida independentemente de seus dispositivos externos, fazendo que com a equipe de desenvolvimento foque no desenvolvimento do requisitos de negócio, ignorando dependências externas técnicas e infra estruturais.

Um mock é componente usado emular ao hexágono os serviços reais ofertados de um dispositivos externo de modo que, o módulo do hexágono possa ser 100% desenvolvido, testado e homologado sem precisar instalar, configurar ou usar serviços, tecnologias ou infra estruturas pendentes a solução.

Exemplos: 
● Classe DAO que ao invés de usar JDBC, faz a persistência em memória usando HashMap.
● Classe DAO que ao invés de usar JDBC, faz a persistência em um sgdb embutido Hsqldb.
● Classe envio de email que ao invés de usar JavaMail, faz System.out.println.

## Fluxo de Execução

Na teoria, queríamos que o core do sistema, o hexágono ficasse isolado e não dependente de ninguém, mas na prática (Runtime), o **hexágono na verdade depende do lado direito**.
Assim, estamos furando a 2º e 3º regra de dependência.

Na prática:
● O centro, o hexágono depende do lado direito.
● Lado direito, os atores secundários não dependem do hexágono.

### Inversão de Controle (Inversion of Control - IoC)

É um padrão arquitetural, uma técnica de arquitetura de software usada para inverter uma linha de dependência em um bloco
arquitetural.
Se o componente A [->depende->] B, usando IoC é possível fazer inverter a seta, fazendo com que A [<- dependa IoC<-] B.
A arquitetura hexagonal aplica IoC, estabelecendo o princípio modular que o **lado de fora direito tem dependência ao hexágono via IoC**!

**Problema resolvido na teoria, na documentação e prática (runtime)**
1. Lado esquerdo, os atores primários dependem do hexágono
2. Lado direito, os atores secundários dependem do hexágono **via IoC**.
3. O centro, o hexágono não depende de ninguém, só dele mesmo.

## Dependências configuráveis

Com a uso de portas polimórficas, IoC e o princípio de flexibilidade de intercâmbio de adaptadores, a arquitetura hexagonal estabelece o uso de um gerenciador de dependências de forma dinâmica e configurável em ambos o lados.

**Lado condutor primário**
A conversa é iniciada pelo driver (ator primário), portanto, o adaptador do driver tem uma dependência configurável na porta do driver, que é uma interface implementada pelo aplicativo.

**Lado dirigido secundário**
A conversa é iniciada pelo aplicativo, portanto, o aplicativo tem uma dependência configurável na porta acionada, que é uma interface implementada pelo adaptador acionado do ator secundário.

Dentro do hexágono
De forma opcional, dentro do hexágono pode haver uma sub organização que pode fazer uso de dependência configurável entre os próprios componentes internos.

Dependência configurável é o princípio mais importante em que se baseia a arquitetura hexagonal, pois permite que o hexágono seja dinamicamente dissociado de qualquer tecnologia de front-end e infra-estrutura. 
E esse desacoplamento é o que possibilita o principal objetivo da arquitetura, ou seja, ter um aplicativo que possa ser executado por vários drivers e testado isoladamente de destinatários / repositórios.

O projeto hexagonal deve fazer uso de qualquer serviços de IoC de sua plataforma de preferência para montar a rede de objetos 
(wiring) de execução da solução, tando os build de desenvolvimento, testes, homologação e produção. Exemplos Java: 
● CDI
● Spring IoC
● Pico Container

## Aplicativo Real - Gerenciador de tarefa








