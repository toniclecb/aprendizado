# package principles

## Os princípios de coesão e acoplamento de módulos

- Coesão: O nível de especificidade de uma classe para desempenhar um papel em um contexto. Relacionado ao princípio da responsabilidade única.
- Acoplamento: Dependência e “conhecimento” entre classes. Relacionado a oquanto uma classe depende da outra para funcionar.
- Granularidade: Corresponde ao nível de detalhamento;

### Os princípios de coesão de módulos

- REP – Release/Reuse Equivalency Principle
- CRP – Common Reuse Principle
- CCP – Common Closure principle

### Os princípios de acoplamento de módulos

- ADP – Acyclic Dependencies principle
- SDP – Stable Dependencies principle
- SAP – Stable Abstractions principle

------------

#### REP – release reuse equivalency principle
Princípio da Equivalência entre o Reuso e a Distribuição

> “a granularidade de reuso não deve ser menor que a granularidade de distribuição”

A reutilização de uma parte de um software é facilitada se ela for separada em uma estrutura apropriada (pacote).
Um critério para agrupar classes é o reuso. Uma vez que pacotes são as unidades da release, eles são também a unidade de reuso.
Código não deve ser reusado com ação de copiar e colar em outra classe.

Esse princípio fala que para se reutilizar componentes de software estes devem ser rastreados por um processo e recebam números de release. Sem esses números não seria possível garantir a compatibilidade dos componentes uns com os outros.

Analisando esse princípio por intermédio da arquitetura de software pode-se concluir que os módulos e classes formados em um componente devem fazer parte de um grupo coeso, de forma que o componente não seja uma mistura de classes e módulos aleatórios, mas sim ter um tema ou propósito que todos os módulos compartilhem.

#### CRP – Common Reuse Principle
Princípio do Reuso Comum

> “classes que não são usadas junto, não devem ser agrupadas”

Se uma classe depende de outra em um pacote diferente, na verdade há uma dependência do pacote inteiro. O agrupamento de classes relacionadas diminui a dependência (acoplamento) entre pacotes.
Uma dependência de um pacote é uma dependência de tudo que está incluído no pacote.
Quando há dependência de um componente, é preciso ter certeza que essa dependência se aplica a todas as classes de um componente, e que as classes de um componente são tão interligadas que se torna impossível depender de uma e não depender de todas. Caso isso não seja aplicado, será preciso implementar novamente mais componentes do que o necessário.

#### CCP – Common Closure principle
Princípio do Fechamento Comum

> “classes que mudam juntas, ficam juntas”

Quanto mais pacotes inter-relacionados há em um projeto, maior o trabalho é para gerenciar, testar e liberar atualizações. O agrupamento de classes que mudarão juntas reduz o número de pacotes modificados.
Um critério para agrupar classes é a partir da previsão de quais irão mudar no mesmo momento.
Esse princípio é uma reformulação do Princípio de Responsabilidade Única (SRP).
O CCP também se relaciona com mais um princípio do SOLID, o Princı́pio Aberto/Fechado (OCP) sendo o termo ”fechado” usado em ambos com o mesmo sentido. O OCP afirma que as classes devem ser fechadas para modificações, mas abertas para extensões.


#### ADP – Acyclic Dependencies principle
Princípio da Dependência Acíclica

> “A estrutura de dependência entre pacotes deve ser um DAG (grafo direto acíclico)”

As dependencias entre os pacotes não devem formar ciclos!
Ciclos podem ser quebrados com o uso do DIP (dependency inversion principle).
Existem dois modos de quebrar esses ciclos (MARTIN, 2017). O primeiro é aplicando o Princípio da Inversão de Dependências, no qual pode ser criada uma interface com os métodos que uma determinada classe precisa e em seguida herdá-la em outra classe, de forma que seja feita a inversão de dependências e assim quebrar o ciclo. O segundo modo é criar um componente que ambas as classes que formam o ciclo dependam, e mover as classes essenciais em ambos para esse novo componente.


#### SDP – Stable Dependencies principle
Princípio da Dependência Estável

> "Dependências devem acontecer do pacote mais instável para o mais estável"

Módulos que são "difíceis de mudar" não devem depender de módulos que são "fáceis de mudar".
Estabilidade não significa baixa frequencia de mudanças, mas significa a quantidade de trabalho para fazer a mudança!
Nem todos os componentes devem ser estáveis, pois isso deixaria o sistema imutável. É necessário projetar a estrutura de componentes de forma a ter componentes estáveis e instáveis.

#### SAP – Stable Abstractions principle
Princípio da Estabilidade Abstrata

> "Pacotes estáveis deveriam ser abstratos."

Pode-se dizer que um pacote é "mais difícil de mudar", pois mais pacotes dependem dele. Portanto, deve ser abstrato para que possa ser estendido quando necessário.
Este princípio estabelece uma relação entre estabilidade e abstração. Ele afirma que um componente estável também deve ser abstrato para que a estabilidade não impeça a sua extensão, e que um componente instável deve ser concreto, posto que sua instabilidade faz com que o seu código concreto seja facilmente modificado. Por consequência, para um componente ser estável ele deve possuir interfaces e classes abstratas de modo que possa ser estendido (MARTIN, 2017).


## Fontes:

http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod

http://wiki.icmc.usp.br/images/1/15/Aula1a-scc266_design_patterns.pdf

https://homepages.dcc.ufmg.br/~jefersson/cursos/dcc052/Aula12.pdf

https://docplayer.com.br/1507792-Padroes-de-projeto-em-desenvolvimento-web-scc-266-prof-renata-pontin-m-fortes-renata-icmc-usp-br-pae-willian-watanabe-watinha-gmail.html

https://web.fe.up.pt/~arestivo/slides/?s=solid#34

https://dev.to/rubemfsv/principios-dos-componentes-o-conceito-por-tras-do-codigo-4529
