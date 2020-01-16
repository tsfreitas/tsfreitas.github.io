---
layout: post
title: "Como monitorar e otimizar o custo na AWS - Parte 1"
categories: 
  - devops
tags:
  - aws
  - cost optimization
---

> "O nosso custo na AWS está muito alto. O que podemos fazer para diminuir ele em 30% ?"

***

Essa é uma frase bem comum quando uma empresa migra de um datacenter clássico para a AWS sem preparar os seus sistemas e processos para trabalhar com a nuvem. Mas como reduzir esse custo e manter uma operação saudável na nuvem?

Um custo baixo na nuvem se dá pela mudança de pensamento para usar os melhores recursos e serviços do provedor escolhido, então não há uma fórmula simples e rápida de manter os custos em ordem, mas sim algumas técnicas que podem ser aplicadas.

Uma delas é entender o quanto cada parte da sua operação impacta na fatura no final do mês.

## Análise de custo

A AWS possui uma ferramenta que ajuda a entender como está a divisão do custo dentro da sua conta, o [_AWS Cost Explorer_][cost_explorer].


![AWS Cost Explorer][cost_explorer_image]


Essa é o visual da ferramenta, sendo ela divida em 3 grandes partes:
- Gráfico e tabela de valores
- Tempo, granularidade (mensal ou diária) e agrupamento
- Filtros (inclusivos e exclusivos)

Com esse recurso conseguimos dividir os custo por serviços, ações na API, regiões e muitas outras visões que nos ajudam a identificar como o custo é dividido.

![Custo por serviços][cost_by_service]

Nesse exemplo os maiores ofensores são os servidores e recursos associados a eles (disco, snapshots, etc). Fácil!
Bem, infelizmente não. Na maioria das vezes esses recursos terão os maiores valores na conta.


> ### Proporção de gastos entre serviços
>
> Em uma operação recém migrada para a AWS é provavel que os sistemas e bancos de dados fiquem em servidores EC2. Uma parte da mudança de pensamento é transferir alguns recursos para os serviços gerenciados.

***

Mesmo sabendo os recursos, as operações ou a região com os maiores gastos, é importante entender como cada sistema e parte da sua operação utiliza esses componentes e impactam no gasto. 

## Tags, tags e mais tags

Uma técnica altamente recomendável é marcar os recursos com tags para identificar a qual parte da sua infra ele pertence.

Mas que tags colocar? Bem não há um número exato mas deve ser o suficiente para identificar qual parte da sua operação aquele recurso faz parte.


> ### A tentação das múltiplas tags
>
> Não existe um número certo de tags para identificar cada recurso, então a tentação é taguear tudo: data de criação, responsável, identificação do ticket de criação do recurso, signo chinês do responsável, etc.
>
> Muitas tags são dificeis de manter, podem com o tempo ter informações desatualizadas e se tornarem mais um ponto de burocracia desnecessária.
>
> Então mantenha o mínimo necessário de tags para identificação do recurso.

***

Um ponto importante é que as tags sejam o mais independente possível de mudanças organizacionais. Então colocar o nome do gerente, o nome do time ou qualquer outra informação que possa ser alterada frequentemente, não é uma boa ideia.

Nas minhas tentativas de marcação de recursos acabei encontrando 3 tags que são independentes e deixam claro as divisões e responsabilidades: *Produto* , *Sistema* e *Ambiente* .
 
Claro que para cada empresa outras tags podem ser melhores. No canal da AWS no youtube existe [um vídeo sobre custo][aws_youtube] mostrando outras possibilidades de tags.

 Mas voltando as 3 tags que utilizo:

### Tag Ambiente

Essa tag informa em qual ambiente o recurso roda, como produção ou desenvolvimento. Recursos do ambiente de desenvolvimento talvez possam ser desligados durante a noite ou podem ter familias de máquinas diferentes.


### Tag Sistema

Essa tag informa a qual sistema o recurso faz parte. Um sistema normalmente possui mais de um elemento, como por exemplo um banco de dados, um servidor de cache, um servidor de aplicação, um balanceador de cargas, etc. Essa tag ajuda a mostrar os diversos elementos pertencentes a um sistema, qual elemento do sistema pode ser melhorado para redução de custo entre outros insights.


### Tag Produto

A maior parte dos recursos criados para a operação, são criados para dar suporte a um produto do negócio. Essa tag é o link entre a infraestrutura e o negócio, ela mostra o custo total que um produto tem dentro da operação. Já vi alguns casos que o custo dos servidores era maior que o lucro que o produto gerava, o que ajudou a área de negócio a decidir pelo seu desligamento.


> ### A tag Name
> Uma boa prática é todos os recursos possuírem a tag Name, essa tag mostra o nome do recurso e, em muitas telas da AWS, são lidas por padrão. Se você ainda não utiliza essa tag, recomendo que comece a utilizar, ela facilitará muito sua vida.

***

## Recursos compartilhados

Há uma certa hierarquia nas tags, todo recurso pertence a um sistema, vários sistemas fazem um produto funcionar e esse produto funciona dentro de um ambiente.

Mas há alguns recursos que são compartilhados entre diversos sistemas e até produtos. 

Recursos de rede, servidores de monitoria, scripts, são alguns exemplos desses recursos compartilhados.

Para tais recursos utilizo alguns valores especiais.
Recursos de rede, por exemplo, são utilizados por todos os produtos, então a tag *Produto* é marcada como *SHARED*. Recursos de monitoria, ganham a tag *Sistema* o valor *MONITORING*. Lambdas de manutenção ganham a tag *Sistema* com o valor *INFRA*.

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

Recursos compartilhados costumam ser ignorados durante o trabalho de redução de gastos, mas eles são detalhes importantes no todo. Uma melhoria em um sistema pode diminuir o consumo de rede, reduzindo o valor de recursos compartilhados ou uma troca de aplicação pode aumentar o meu consumo de dados da internet. Sempre é importante ter um olho atento a esses recursos.

## Recursos desconhecidos

É possível que certos recursos existentes não sejam tão conhecidos. Aquele bucket que guarda imagens pertence a qual Produto? Esse servidor de banco de dados é usado por qual Sistema?

Caso perguntas assim apareçam durante o processo de tagueamento, uma alternativa é marcar essas recursos com o valor *DESCONHECIDO*. 

Assim o bucket de imagem teria a tag *Produto: DESCONHECIDO*; o servidor de banco de dados teria a tag *Sistema: DESCONHECIDO*.  Essa marcação identifica que o recurso deve ser visto posteriormente.

O importante a se lembrar no processo de tagueamento é que o ideal é que todos os recursos estão marcados, mesmo que como *DESCONHECIDO*. Isso mostra que você pelo menos já viu a cara daquele recurso, que ele está na luz.

## Case sensitive

Os nomes e valores das tags diferenciam letras maiúsculas de minúsculas, assim *web-server* e *WEB-SERVER* são vistos como valores diferentes, dificultado as análises.

Por isso sempre assumo que nomes de tags são em PascalCase e os valores são em maiúsculo com hífens no lugar dos espaços.

## Ativando tags para custo

Para que as tags sejam vistas no AWS Cost Explorer é necessário ativar seu processamento na parte de Billing do console. 

A ativação é bem simples: vá em *Cost allocation tags* selecione as tags desejadas e clique em Ativar, as tags podem levar até 24 horas para começarem a ser exibidas no Cost Explorer. Tags já ativas são mostradas no Status.

![Tag Allocation][tag_allocation_image]

Mais detalhes podem ser vistos na [documentação][tag_allocation].


## Edição de tag em massa

Um problema da marcação de recursos é saber de forma fácil quais recursos possuem ou não tags. Para isso a AWS disponibiliza o [Tag Editor][tag_editor].


Essa ferramenta permite ver todos os recursos e suas tags, podendo inclusive filtrar quais recursos não possuem a tag específica.

![Tag Editor][tag_editor_image]

Essa ferramenta não mostra apenas os recursos e suas tags, também é possível fazer a edição em massa desses recursos. Com essa feature é possível começar rapidamente o tagueamento de recursos e já ter uma ideia de como está a divisão dos gastos.

![Tag Editor Multiple Resources][tag_editor_multiple_resources_image]

## Analisando custo com tags

Tendo as tags configuradas é possível entender como o custo está divido.

![Recursos Tagueados][tagged_resources_image]


Assim, olhando o exemplo acima é possível ver que diminuindo o custo do MAIN_DB ou do WEB_APP conseguimos reduzir a conta no final do mês. 

Bem, não é tão fácil assim.

Toda arquitetura possui gargálos de custos: recursos que são peças mais caras no processo mas são necessárias e, por mais que possa parecer óbvio atacar essas partes, olhar os sistemas periféricos pode ser a alternativa mais rápida para diminuir a conta.

Então analisar se é possível melhorar o MICROSERVICE_X ou o TEST_NVME pode ser mais rápido do que ver como melhorar o custo do MAIN_DB.

> ### As partes ocultas do custo
>
> Ao olhar o custo de um sistema é comum focar no servidor, sendo tentando trocar a máquina, sendo desligando ela durante alguns períodos. Esse ponto é importante mas vale olhar outros tipos de recursos, como disco, snapshots, rede, elastic ip e balanceadores de carga. 

## Conclusão

Ao taguear os recursos é possível ter uma nova visão da divisão de custo e de utilização da arquitetura. Essa é a base para um plano de redução e evolução da arquitetura para se adaptar à nuvem.

As tags também são importantes para ligar os recursos de infraestrutura com o negócio em si. Em muitos lugares o custo do taxi para visitar clientes é mais contabilizado do que o custo para rodar os sistema que o  cliente usa.

Após a identificação de custo é possível caminhar para outra parte da melhoria de custo: o rightsizing. Mas isso fica para outro post ;)


[cost_explorer]: https://aws.amazon.com/aws-cost-management/aws-cost-explorer/
[aws_youtube]: https://youtu.be/qd84dbS7L5M
[tag_allocation]: https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/activating-tags.html
[tag_editor]: https://docs.aws.amazon.com/ARG/latest/userguide/tag-editor.html


[cost_explorer_image]:  /assets/aws-cost-part1/cost_explorer.png
[cost_by_service]:      /assets/aws-cost-part1/cost_by_service.png
[tag_allocation_image]: /assets/aws-cost-part1/tag_allocation.png
[tag_editor_image]: /assets/aws-cost-part1/tag_editor.png
[tag_editor_multiple_resources_image]: /assets/aws-cost-part1/tag_editor_multiple_resources.png
[tagged_resources_image]: /assets/aws-cost-part1/tagged_resources.png
