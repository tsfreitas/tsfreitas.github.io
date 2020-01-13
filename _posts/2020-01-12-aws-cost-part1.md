---
layout: default
title: "Como monitorar e otimizar o custo na AWS - Parte 1"
categories: 
  - devops
tags:
  - aws
  - cost optimization
---

> "O nosso custo na AWS está muito alto. O que podemos fazer para diminuir ele em 30% ?"

Essa é uma frase bem comum quando uma empresa migra de um datacenter clássico para a AWS sem preparar os seus sistemas e processo para trabalhar com a nuvem. Mas como reduzir esse custo e manter uma operação saudável na nuvem?

Um custo baixo na nuvem se dá através da mudança de pensamento da operação para usar os melhores recursos e serviços do provedor escolhido, então não há uma fórmula simples e rápida de manter os custos em ordem, mas sim algumas técnicas que podem ser aplicadas.

Uma delas é entender o quanto cada parte da sua operação impacta na fatura no final do mês.

## Análise de custo

A AWS possui uma ferramenta que ajuda a verificar como estão os gastos da sua conta, ela se chama [_AWS Cost Explorer_][cost_explorer]

![AWS Cost Explorer][cost_explorer_image]

Essa é o visual da ferramenta, sendo ela divida em 3 grandes partes:
- Gráfico e tabela de valores
- Tempo, granularidade (mensal ou diário) e agrupamento
- Filtros (inclusivos e exclusivos)

Com esse recurso conseguimos dividir os custo por serviços, ações na API, regiões e muitas outras visões que nos ajudam a identificar como o custo é dividido.

![Custo por serviços][cost_by_service]

Então os nossos maiores oferensores são os servidores e recursos associados a eles (disco, snapshots, etc). Fácil!
Bem, infelizmente não. Na maioria das vezes esses recursos terão os maiores valores na conta.

> ### Proporção de gastos entre serviços
>
> Em uma operação recém migrada para a AWS é provavel que os sistemas e bancos de dados fiquem em servidores EC2. Uma parte da mudança de pensamento é transferir alguns recursos para os serviços gerenciados.

## Tags, tags e mais tags

Uma técnica altamente recomendável é marcar os recursos com tags para identificar a qual parte da sua infra ele pertence.

Mas que tags colocar? Bem não há um número exato mas deve ser o suficiente para identificar qual parte da sua operação aquele recurso faz parte.


> ### A tentação das múltiplas tags
>
> Não existe um número certo de tags para identificar cada recurso, então a tentação é taguar tudo: data de criação, responsável, identificação do ticket de criação do recurso, signo chinês do responsável
>
> Muitas tags são dificeis de manter, podem com o tempo ter informações desatualizadas e se tornarem mais um ponto de burocracia desnecessária.

Já trabalhei tagueando recursos em algumas oportunidades e achei o número mágico de 3 tags:
 - Produto
 - Sistema
 - Ambiente

 No canal da AWS no youtube existe [um vídeo sobre custo][aws_youtube] mostrando outras possibilidades de tags


### Tag Ambiente

Esta tag informa em qual o ambiente o recurso roda, como produção ou desenvolvimento. Recursos do ambiente de desenvolvimento talvez possam ser desligados durante a noite ou podem ter familias de máquinas diferentes.


### Tag Sistema

Esta tag informa a qual sistema o recurso faz parte. Um sistema normalmente possui mais de um elemento, como por exemplo um Banco de dados, um servidor de cache, um servidor de aplicação, um balanceador de cargas, etc. Essa tag ajuda a mostrar os diversos elementos pertencentes a um sistema, qual elemento do sistema pode ser melhorado para redução de custo entre outros insights.


### Tag Produto

A maior parte dos recursos criados para a operação, são criados para dar suporte a um produto do negócio. Esta tag é o link entre a infraestrutura e o negócio, ela mostra o custo total que um produto tem dentro da operação. Já vi alguns casos que o custo dos servidores era maior que o lucro que o produto gerava, o que ajudou a área de negócio a decidir pelo seu desligamento.


> ### A tag Name
> Uma boa prática é todos os recursos possuírem a tag Name, essa tag mostra o nome do recurso e em muitas telas da AWS, são lidas por padrão. Se você ainda não utiliza essa tag, recomendo que comece a utilizar, ela facilitará muito sua vida.

## Recursos compartilhados

Há uma certa hierarquia nas tags, todo recurso pertence a um sistema, vários sistemas fazem um produto funcionar e esse produto funciona dentro de um ambiente.

Mas há alguns recursos que são compartilhados entre diversos sistemas e até produtos. 

Recursos de rede, servidores de monitoria, scripts, são alguns exemplos desses recursos compartilhados.

Para tais recursos utilizo alguns valores especiais.
Recursos de rede, por exemplo, são utilizados por todos os produtos, então a tag *Produto* desse recurso é marcada como *SHARED*. Recursos de monitoria, ganham na tag *Sistema* o valor *MONITORING*. Lambdas de manutenção ganham a tag *Sistema* cm o valor *INFRA*.

Assim os recursos compartilhados sempre ficam com as tags:
- Ambiente:
  - PRODUCAO -  se o recurso for utilizado por pelo menos um outro recurso de produção;
  - DESENVOLVIMENTO - se o recurso for utilizado apenas por outros recursos de desenvolvimento;
- Sistema:
  - INFRA - se o recurso for de um recurso genérico de infraestrtura;
  - MONITORING - se o recurso for utilizado para monitoramento;
- Produto:
  - SHARED - recurso utilizado por diversos produtos;
  - MONITORING - recursos utilizados apenas para monitoria;

Recursos compartilhados costumam ser desprezados durante o trabalho de redução de gastos, mas eles devem ser monitorados e contabilizados nos trabalhos. Uma melhoria em um sistema pode diminuir o consumo de rede, reduzindo o valor de recursos compartilhados. Sempre é importante ter um olho atento a esses recursos.

## Case sensitive

Os nomes e valores das tags diferenciam letras maiusculas de minusculas, assim *web-server* e *WEB-SERVER* são vistos como valores diferentes, dificultado as análises.

Por isso sempre assumo que nomes de tags são em PascalCase e os valores são em maisculo com hífens no lugar dos espaços.

## Ativando tags para custo

Para que as tags sejam vistas no AWS Cost Explorer é necessário ativar seu processamento na parte de Billing do console. 

A ativação é bem simples, selecione as tags desejadas e clique em Ativar, as tags podem levar até 24 horas para começarem a ser exibidas no Cost Explorer. Tags já ativas são mostradas no Status

![Tag Allocation][tag_allocation_image]

Mais detalhes podem ser vistos na [documentação][tag_allocation].


## Edição de tag em massa

## Analisando custo com tags




[cost_explorer]: https://aws.amazon.com/aws-cost-management/aws-cost-explorer/
[aws_youtube]: https://youtu.be/qd84dbS7L5M
[tag_allocation]: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/activating-tags.html


[cost_explorer_image]:  /assets/aws-cost-part1/cost_explorer.png
[cost_by_service]:      /assets/aws-cost-part1/cost_by_service.png
[tag_allocation_image]: /assets/aws-cost-part1/tag_allocation.png
