# Introdução a Arquitetura Hexagonal

https://www.udemy.com/course/introducao-a-arquitetura-hexagonal/learn/lecture/15421460

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



