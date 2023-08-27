#### 24/08/2023

Curso de NGINX Parte 2: performance, FastCGI e HTTPS

```
Utils Command:

sudo lsof -i:8080
kill -9

brew services start jenkins-lts
brew services stop jenkins-lts
brew services restart jenkins-lts

php -S localhost:8000

nginx -s reload
nginx -t

php -S localhost:8000

brew install nginx
brew install php
```

@01-Load balancer

@@01
Apresentação

[00:00] Olá, pessoal! Boas-vindas à Alura. Eu sou o Vinicius Dias e vou guiar vocês nesse segundo treinamento de nginx. No treinamento anterior, nós já aprendemos o que é nginx, para que serve, como configurar um servidor web, proxy reverso, um load balancer, e agora vamos nos aprofundar em todos esses conteúdos.
[00:16] Vamos ver mais algoritmos de load balancer, vamos entender o que é esse tal de round Robin, vamos ver sobre o least connections, waited round Robin, vamos entender todos esses nomes, e depois vamos conhecer um tal de fast cdi, como a internet evoluiu até chegar nesse ponto e o que mais existe, além disso. Mas claro, na prática, vamos fazer requisições para um servidor de fast cdi.

[00:41] Depois vamos falar um pouco de performance. Principalmente usando o que o http já fornece para nós, cache, HTTP, compressão, keep alive, ou seja, manter conexões abertas, esse tipo de coisa bastante importante e interessante. Depois vamos falar um pouco sobre cache no lado do servidor, porque o nginx além de ser um load balancer, um proxy reverso, um servidor web, também pode ser um servidor de cache, e vamos ver como implementar isso, na prática.

[01:06] No final vamos dar uma pincelada no assunto de segurança, configurando nginx para ler nossos certificados gerados por nós mesmos para abrir um site como https. Nós vamos entender um pouco sobre esse monte de nome de entidades certificadoras, certificados digitais, etc., mas nós vamos utilizar um certificado auto assinado para não usar nada externo e para não precisarmos de domínios, etc.

[01:30] Mas não se assuste que isso tudo vai ficar bem mais claro lá para a frente. E se nesse processo alguma dúvida surgir, não hesite, abra um tópico no fórum, eu tento responder pessoalmente sempre que possível, mas quando não consigo nossa comunidade de alunos, moderados e instrutores é muito solícita e com certeza alguém vai conseguir te ajudar. Mais uma vez seja bem-vindo, espero que você aproveite bastante esse treinamento e até o próximo vídeo para começarmos a colocar a mão na massa.

@@02
Definindo pesos

[00:00] No final do último treinamento, o que nós fizemos? Nós implementamos essa ilustração, um load balancer, ou um balanceador de cargas. Vamos recapitular o que fizemos, como isso foi implementado para pensarmos em um possível problema.
[00:18] Temos um cliente, você, eu, qualquer pessoa na internet, acessando algum site, imagino que é o site da Alura. E o site da Alura por receber muitas requisições precisamos separar em mais de um servidor. Caso um servidor caia temos outro de pé, funcionando, para não sobrecarregar esses servidores, porque eles não aguentariam toda essa carga. Por vários motivos precisamos dividir esses servidores.

[00:45] Quando esse cliente acessa um site da Alura temos o engine next configurado como um load balancer e cada requisição vai cair em um servidor diferente. Chegou a primeira requisição? Cai no primeiro. Chego outra requisição? Cai no segundo. Chegou mais uma? Primeiro, segundo, e vai dividindo igualmente as requisições.

[01:04] Se temos 100 requisições chegando, 50 vão chegar para um servidor, 50 vão chegar para o outro. Vamos só recapitular rápido como implementamos isso. Vou abrir meu terminal. Lembrando que como estou no Mac meus caminhos são esses, mas para você saber o caminho na sua configuração, no seu sistema operacional, nginx -t vai mostrar o arquivo padrão.

[01:25] Então meu arquivo padrão é esse nginx conf, nesse arquivo padrão sei que ele inclui todos os arquivos .conf de servers. Temos um load balancer, vamos abrir ele para ver e se precisar já modificar. Então usr/local/etc/nginx/servers/load-balancer, nesse load balancer estamos ouvindo a porta 8003 e cada hora que chega uma requisição para essa porta hora ele vai mandar para o serviço 1 hora ele vai mandar para o serviço 2.

[02:00] Vamos testar isso de novo só para relembrarmos, hora ele manda para o serviço 1, e quando atualizo ele vai para o serviço 2. Ele vai alternando entre um e outro.

[02:22] Recapitulamos o conceito de load balancer, vamos avançar um pouco. Imagine que por algum motivo, porque nossa infraestrutura é assim, porque isso foi configurado dessa forma, por algum motivo temos um servidor maior do que outro, com mais capacidade de responder requisições.

[02:38] Imagine que nós temos dois servidores respondendo, no nosso caso o de serviço 1 e o de serviço 2. E o servidor que contém esse serviço 1, que está respondendo nessa porta, tem o dobro de memória RAM, de CPU, o dobro da capacidade de receber requisições. Então se ficamos mandando metade para cada um, se mantemos essa estrutura de chega uma requisição vai para um, chega outro vai para outro, vamos acabar sobrecarregando esse servidor menor ou não utilizando todos os recursos disponíveis no servidor maior.

[03:12] O que quero fazer é poder atribuir pesos diferentes para cada um dos meus servidores disponíveis, e fazer isso com nginx é muito simples, posso informar que este servidor 1 tem o peso de 2. Posso usar qualquer número, posso dizer que tem peso 5 enquanto o outro tem peso 2, ou seja, a cada 7 requisições que chegam no meu sistema 5 vão cair no serviço 1 e duas vão cair no serviço 2.

[03:42] Posso configurar esse peso de forma proporcional. Vou manter configurado o serviço 1 com peso 2 e o serviço 2 vai estar no padrão dele, que é o peso 1. Vou salvar isso, mas lembrando, se alterei essa configuração preciso fazer o meu reload. Nginx recarregado sem erro nenhum, vamos ver o que vai acontecer quando faço as requisições.

[04:10] Caiu uma requisição no serviço 1, recarreguei mais uma requisição no serviço 1, recarreguei agora no serviço 2. Quando recarrego de novo, serviço 1, serviço 1, serviço 2. A cada duas requisições no serviço 1, uma requisição cai no serviço 2. Dessa forma conseguimos balancear melhor nossa carga. E esse é um dos algoritmos possíveis para balancear a carga. Então vamos falar um pouco de teoria já que já vimos a prática no próximo vídeo.

@@03
Servidores diferentes?

Vimos neste vídeo que existe a possibilidade de atribuirmos pesos diferentes para cada servidor, fazendo com que as conexões enviadas para cada sejam distribuídas proporcionalmente.
Em que cenário faz sentido atribuirmos pesos diferentes a cada servidor?

Alternativa correta
Quando temos servidores com capacidades diferentes.
 
Alternativa correta! Se o servidor A possui o dobro de recursos (como RAM e CPU) do que o servidor B, talvez faça sentido enviar o dobro de requisições para ele. Para esses cenários, usamos o peso.
Alternativa correta
Nunca devemos ter pesos diferentes para cada servidor.
 
Alternativa correta
Quando quisermos desativar algum servidor, dando peso 0.

@@04
Algoritmos de balanceamento

[00:00] Boas-vindas de volta. No último vídeo nós implementamos uma nova forma de um balanceamento de carga. Nós incrementamos nosso conhecimento de balanceamento de carga. Mas nós estudamos muita prática e não falamos tanto da teoria, então vamos entender a teoria por trás de algoritmos de balanceamento de carga.
[00:18] Por padrão quando configuramos aquele upstream e definimos um proxy pass para um upstream nós temos essa figura, essa é a implementação padrão de um load balancer com nginx. No último vídeo aprendemos a definir pesos, para chegar em algo próximo desse cenário, onde quando tenho várias requisições posso mandar mais requisições para um servidor do que para o outro.

[00:42] Essa ideia nós já entendemos na prática, mas, na teoria, esse algoritmo tem um nome, e é um nome importante de conhecermos, até porque esse mesmo algoritmo é utilizado em cenários fora de balanceamento de carga, então o nome desse algoritmo é round Robin, ele é utilizado inclusive em contextos de escalonamento de processos.

[01:02] Não vou entrar em detalhes sobre o que é escalonamento de processos, mas imagina que tenho vários processos para serem executados na mesma CPU, no mesmo núcleo da CPU, como o processador vai decidir qual processo executar por tanto tempo? A implementação de round Robin define um espaço de tempo onde o processo vai ser executado, depois interrompido.

[01:25] Esse mesmo espaço de tempo, esse mesmo período vai ser executado por outro processo, esse mesmo período vai ser entregue para outro processo executar. Depois outro, depois outro, e assim ele vai rodando. Quando o último finalizou eu volto para o primeiro. Essa é a ideia por trás de um round Robin.

[01:42] Quando tenho essa mesma ideia implementada em um load balancer tenho isso que o nginx traz por padrão. Se tenho três servidores, por exemplo, imagine que tenho mais um, vou fazer a ligação. Se tenho três servidores a primeira requisição chega nesse, a segunda no segundo, a terceira no terceiro.

[02:02] Acabou a lista de servidores nós voltamos para o começo, por isso esse nome round Robin, então vamos em quarta requisição, quinta requisição, sexta requisição. Esse é o algoritmo round Robin, só que quando precisamos definir pesos diferentes para cada um dos servidores, por exemplo, temos o que é chamado de waited round Robin ou round Robin ponderado.

[02:22] Então atingimos esse objetivo através da diretiva wait, demos um peso maior para o servidor, para o primeiro servidor, e o segundo servidor tem um peso um pouco menor. A primeira requisição chega aqui, a segunda também, a terceira chega aqui, a quarta chega aqui, a quinta também, a sexta chega aqui. Assim vamos balanceando, baseado nesse espaço que temos, no número de requisições, puramente. Número de requisições que chegam.

[02:52] Mas existem outros algoritmos que podem fazer sentido em outros casos, não vou citar todos, para não nos complicarmos muito, mas vou mostrar mais dois aqui. Imagine que vai ser uma requisição, uma conexão aberta. A primeira requisição chegou aqui, chegou a segunda requisição, e a terceira. A quarta chegou, a quinta chegou também, e a sexta chegou embaixo.

[03:22] Mas o que acontece? Essa primeira requisição era para uma coisa muito simples, então ela já foi encerada, a terceira também. E vamos receber mais uma requisição embaixo, repare que nesse momento estou mandando mais requisições para o meu servidor mais fraco do que para o servidor mais potente, porque minha única métrica é a quantidade de requisições que estão chegando, e não a quantidade de requisições que estão abertas, ou seja, o número de conexões que meu servidor tem abertas.

[03:55] Nesse cenário, quando tenho requisições que podem ter comportamentos diferentes, onde posso ter requisições muito demoradas e requisições muito simples precisamos balancear isso de forma mais inteligente. Vou apagar isso tudo e vamos pensar em outro cenário.

[04:11] Vou manter só uma requisição, chegou a primeira requisição. Quando chega a segunda, o load balancer sabendo daqueles pesos que tenho que ter o dobro de requisições vai ver qual servidor tem menos conexões baseado naquele peso. Esse servidor pode receber uma conexão a mais porque ele ainda não tem o dobro lá, vou receber mais uma requisição.

[04:33] Chegou a próxima requisição, ele vai ver, opa, esse servidor tem duas conexões ativas, e esse outro nenhuma. Vou mandar para cá. E agora nesse meio tempo uma das requisições já foi finalizada. Essa ainda está ativa. Então o load balancer vai mandar mais uma para cá, e de novo temos só duas conexões ativas, então ele pode mandar a próxima para baixo, ele pode mandar mais uma para cá, porque ainda não tem o dobro.

[05:04] Quando essa requisição morreu, a próxima requisição pode vir para cá, porque tenho três em cima e nenhuma aqui. Ele vai balanceando de forma mais inteligente, sem sobrecarregar nenhum dos dois. Talvez isso tenha ficado muito complexo no desenho, mas basicamente é com peso ou sem peso ele vai ver quais servidores têm menos conexões e mandar essa requisição para lá.

[05:25] Vamos pensar nisso sem peso, para ficar um pouco mais simples. Chegou uma requisição, vamos trazer para cá. Essa requisição chegou aqui, quando chega a próxima requisição o load balancer vai ver qual tem menos requisições, conexões ativas, ele vai mandar para baixo, e agora qual tem menos conexões ativas? Está igual. Então começa de novo, vai lá para o primeiro servidor.

[05:50] Nisso, qual tem menos requisições ativas? O segundo, mandou para cá. Porém, enquanto isso, esse servidor a primeira requisição já foi respondida, quando mando a próxima requisição ele pode mandar para cá de novo, balanceando melhor. E nessa meio tempo essa requisição, essa conexão foi encerrada, então posso mandar a próxima para cá. Ele vai balanceando de forma mais inteligente.

[06:15] Essa é a ideia por trás desse algoritmo de list connections, ou menor número de conexões. E para implementar ele é bastante simples. Basta eu informar no início do meu upstream essa diretiva least_comn, com isso ele vai ver qual o servidor, qual dos meus servers tem menos conexões ativas, abertas, e vai mandar requisição para lá, e ele considera o peso.

[06:42] Segundo essa informação, again with server wait taken into consideration, então de novo com peso dos servidores sendo considerado. Aqui já temos dois algoritmos, o round Robin, que é o padrão, que já vem implementado sem informarmos nada, e nós temos o de least connections, o que podemos configurar se tivermos essa diretiva least connections.

[07:07] Existem vários outros, só vou passar o olho, por exemplo, IP Hash, então baseado no hash do ip, ou seja, nas informações do ip que chegou à conexão posso mandar sempre esse mesmo ip para o mesmo servidor. Dessa forma consigo manter uma seção ativa, informações stateful lá, isso não é tão comum assim, mas pode ser bastante útil em alguns cenários.

[07:30] Tenho um generic hash, posso ter alguma informação gerando hash, então sempre que cair nessa url vai para esse servidor. Quando cair nessa outra url vai para outro servidor. Posso ter informações assim, também é menos comum. Posso ter com least time, então para cada requisição o nginx pode ver qual servidor está com a menor latência além de menor número de conexões, então esse já é um pouco mais complexo, ele é utilizado apenas no nginx plus, ou seja, na versão paga.

[08:05] Repare que esses outros não têm essa observação, então usamos na versão open source, e o último que pode ser simples de entender, mas que pode trazer complexidade é o de random, ou seja, cada requisição vai ser mandada para um servidor selecionado de forma aleatória.

[08:22] Mas além dessa ideia aleatória nós podemos trazer algumas configurações. Por exemplo, de todos os servidores, tenho oito servidores, seleciono aleatoriamente dois, e dentro desses dois podemos distribuir através do menor número de conexões, do least time, etc.

[08:40] Podemos configurar as coisas de formas muito complexas, mas te garanto que os mais comuns são o round Robin e o waited round Robin, e o least connections, quando usamos um desses dois se todas as nossas requisições vão para um servidor de aplicação ou se todas as nossas requisições só servem conteúdos estáticos, o round Robin é perfeito, nós não precisamos de nada além disso.

[09:04] Agora, se algumas conexões, se um dos meus servidores é maior que outro, tem mais recursos, então o waited round Robin já resolve o caso. Agora se tenho requisições, conexões que vão demorar para ser respondidas e outras que são muito rápidas, então o algoritmo de least connections pode ajudar bastante. Eu posso ter muitas conexões demoradas em um servidor e passo a mandar mais requisições para o outro, dessa forma temos um rebalanceamento um pouco mais esperto.

[09:33] Então se esse é um pouco mais esperto, por que eu utilizaria o round Robin? Porque ele exige menos do nosso load balancer, ele recebe a requisição e simplesmente vê, o último que mandei foi o servidor 2? Então, agora é 1. O último foi 1? Então, agora é 2. Já esse de least connections precisa armazenar quantas conexões estão abertas, ele tem um custo a mais. Depende do cenário onde você vai implementar cada um, mas essas são as características, prós e contras.

@@05
Round-robin vs Least_conn

Vimos alguns diferentes algoritmos de load balancing mas os 2 em que focamos foram Round-Robin e Least Connected.
Qual a diferença prática entre ambos?

least_conn vai verificar quantas conexões há abertas em cada servidor.
 
Alternativa correta! Este algoritmo verifica quantas conexões há abertas com cada servidor para decidir para onde enviar a próxima requisição. Já o round-robin simplesmente armazena para onde foi enviada a última conexão.
Alternativa correta
round-robin vai verificar quantas conexões há abertas em cada servidor.
 
Alternativa correta
Ambos são sinônimos.

@@06
Servidores de backup

[00:00] Boas-vindas de volta. Já entendemos alguns algoritmos de balanceamento de carga, até aplicamos o Weighted round Robin, mas agora quero trazer outro conceito bastante interessante, que um servidor de carga pode nos ajudar. Imagine que eu tenha um servidor ou um grupo de servidores e esses servidores estão realmente prontos para receber as requisições, eles foram feitos para isso, estão configurados direto, tem todas as otimizações. E tenho um ou um grupo de servidores que estão disponíveis para mim, mas não são os melhores para executar aquela tarefa.
[00:35] Eles estão disponíveis, eu posso utilizar eles, mas eles não são os mais performáticos, vamos dizer assim. Mas no cenário do meu servidor, do meu grupo de servidores principais falhar, ou seja, acontecer alguma pane, quero poder utilizar esse servidor mais fraco ou não tão otimizado como backup.

[00:54] Outro cenário que também pode ser utilizado. Imagina que tenho um servidor, que é um servidor de banco de dados, ou um servidor de arquivos, ou algo que não recebe muita requisição, um servidor interno da empresa, só que quando meu servidor real cair, se acontecer alguma pane, algo muito sério que torne minha aplicação indisponível, eu posso utilizar essa minha máquina, que é um servidor de arquivos, um servidor que não recebe tanta requisição como um servidor web de backup.

[01:22] Então o que vamos fazer aqui é transformar nosso serviço1 como servidor principal, ou seja, ele vai ser o que sempre recebe as requisições, mas se esse serviço1 não estiver mais disponível quero mandar as requisições para o serviço2, ou seja, meu serviço2 agora vai agir como um servidor de backup.

[01:42] Vamos nas configurações do meu load balancer, tirei aquela configuração de weight, mas você pode manter ela se quiser sem problema nenhum, para o que vamos fazer aqui vai funcionar. Mas o que vou fazer agora é informar, esse serviço2 agora é um servidor de backup, por isso que tirei o weight porque não faria sentido, como só tenho um servidor disponível não faz sentido ficar dando peso para ele.

[02:05] Mas de novo, o que estou fazendo aqui? O meu servidor 1 agora é meu único servidor principal, meu servidor real, mas se esse servidor 1 falhar, o 2 vai ser utilizado, porque ele é um servidor de backup. Vou salvar e recarregar.

[02:25] Agora repara que todas as requisições que eu fizer vão cair no serviço 1. Estou atualizando e tudo vai cair no serviço 1, mas agora se eu for lá na minha configuração de micro serviços e fizer com que isso não exista mais, por exemplo, vou mudar da porta 8001 para 8004.

[02:44] Como meu servidor 8001, meu serviço 1 não existe mais, se eu salvar isso e recarregar, o que acontece? Recarreguei, o nginx faz algumas verificações, garante que todos os servidores estão de pé, etc. Ele já reparou que aquele meu serviço 8001 não existe mais. Aquele servidor que meu load balancer vai tentar fazer requisição, meu upstream já verificou que isso está indisponível.

[03:14] Ou seja, quando eu fizer uma requisição agora, vou cair no meu serviço 2, que é meu serviço de backup, então quando eu fizer requisição, caio no serviço 2. Repare que sempre que atualizo, isso acontece. Mas Vinicius, em um cenário real eu não vou ficar atualizando. Então como o nginx vai saber que meu serviço está disponível ou indisponível?

[03:36] Existem dois cenários, o padrão, o que vai acontecer e o que já está acontecendo como está configurado, e o cenário quando temos a licença comercial do nginx que podemos configurar. Então vamos entender o que acontecendo e só passar o olho, só falar rapidinho sobre o que podemos atingir com a licença comercial.

[03:54] Por padrão sempre que reinicio o nginx ele garante que meus upstream, que meus servidores existem, pelo menos. Ele não faz nenhuma requisição complexa, não faz nenhuma verificação complexa, mas ele garante que esse servidor está respondendo alguma coisa.

[04:11] Agora, se durante o uso um servidor, o serviço 1 caiu, por exemplo, o nginx vai mandar a requisição para o serviço 1 e ele vai receber uma falha, então ele vai identificar esse serviço 1 como indisponível. Então preciso ir lá e tratar isso. De tempos em tempos que posso configurar o nginx vai tentar passar uma requisição para lá de novo, então tenho esse período para consertar esse servidor, por exemplo.

[04:40] E obviamente isso é configurável. Posso vir no max_fails, o número máximo de falhas que meu serviço pode ter antes de ser considerado indisponível. Com isso posso informar, se meu serviço recebeu três requisições e não respondeu, três requisições com falha, considera ele indisponível. Por fail timeout.

[05:11] Vamos dar uma olhada na descrição do fail timeout, ele define o tempo em que vou esperar entre uma requisição e outra falhar para considerar que isso realmente é um servidor indisponível e também o período em que esse servidor vai ser considerado indisponível.

[05:30] Ficou essa primeira parte um pouco confusa, então basicamente é se estou dizendo que três falhas indicam que meu servidor está indisponível, recebi uma requisição, daqui dez segundos recebo outra, daqui mais dez segundos recebo outra, se essas três chegaram, se dentro desses dez segundos as três falharam, considero esse servidor indisponível.

[05:55] Mas, por padrão, esse max fails é 1. Então é o comportamento desejado na maioria dos casos. Ou seja, falhou uma vez esse fail timeout vai simplesmente identificar, informar o período em que esse servidor vai ser considerado indisponível. Vamos simplificar um pouco.

[06:15] Imagine que eu queira se um servidor receber uma requisição ele vai ser identificado como indisponível, isso já é o padrão, e quero que esse servidor só receba requisições de novo depois de um minuto, então posso ir em fail timeout e definir como 60 segundos, por exemplo.

[06:36] Repare que temos 30 segundos, posso colocar 60 segundos em fail timeout. Ou se quero dois minutos posso ter 120 segundos. Dessa forma o que acontece? Meu servidor caiu, enquanto está rodando, as coisas estão funcionando, esse serviço 1 não está mais disponível, ele é identificado como indisponível e as requisições chegam para o meu serviço 2. Porém, nesse meio tempo não quero deixar o meu serviço 2 recebendo todas as requisições, porque ele é um serviço de backup, ele não está pronto para receber todas as requisições, ele é um pouco mais fraco.

[07:20] Estou me dando esse tempo para descobri o que deu de errado e colocar esse serviço no ar de novo. Então depois de, por exemplo, dois minutos esse serviço vai voltar a receber requisições, então se eu não tiver resolvido ainda algum cliente pode receber um erro.

[07:35] Essa é a ideia por trás do fail timeout e se eu quiser posso receber várias requisições antes de considerar isso como indisponível. Pode ser um problema, mas em alguns casos pode ser útil, então podemos utilizar o max fails. Por padrão é um, e nunca vi honestamente um caso em que precisamos receber várias requisições para considerar isso como indisponível.

[07:58] Esse é um cenário padrão e como comentei tem um cenário que o uso comercial permite que nós façamos, que é o módulo de help check, que é a verificação automática, já informo faça nginx você mesmo essa verificação, se meu serviço está de pé ou não, sem ninguém precisar fazer uma requisição ou algo do tipo, através dessa diretiva.

[08:24] Tenho o health check e se eu tentar adicionar isso aqui, por exemplo, dentro de algum location, ou seja, algum servidor que existe, posso definir para o nginx fazer algumas requisições para cá de tempos em tempos. Quero ver se meu serviço 1, por exemplo, que vou voltar ele aqui, está tudo certo, tudo ok, posso adicionar o health check, porém, quando tento fazer o nginx reload vou ter um problema.

[08:58] Ele vai informar que essa health check não existe, porque estou utilizando a versão open sem essa diretiva habilitada. Então no uso comercial podemos dizer nginx, de tempos em tempos faz uma requisição para o meu serviço, verifica você se ele está de pé, porque dessa forma um usuário talvez nem bata, nem receba o erro. O nginx vai encontrar esse erro antes.

[09:22] Essa é a ideia por trás de servidores de backup, como eles funcionam na prática e como o health check do serviço comercial pode nos ajudar. Como não estamos usando essa versão comercial, vou remover e fazer nosso reload de novo e garantir que temos o load balancer mais uma vez mandando para os dois serviços. Mas agora temos um serviço de backup.

[09:45] Nosso serviço 1 vai sempre receber as requisições e caso ele caia nosso serviço 2 vai ser o de backup, dois minutos depois o serviço 1 pode receber requisições de novo, pode tentar receber requisições de novo, essa é a ideia por trás de serviços de backup, como um load balancer pode nos ajudar em cenários de pane, mas acho que já vimos bastante coisa de load balancer, então vamos mudar um pouco o foco do estudo, vamos conhecer outras possibilidades do nginx no próximo capítulo.

@@07
Por que usar?

Vimos que existe a possibilidade de marcar um servidor como backup, fazendo uso dele apenas em casos de pane.
Por que marcaríamos um servidor como backup ao invés de usá-lo ativamente?

Não existe motivo para usarmos um servidor como backup.
 
Alternativa correta
Porque assim podemos concentrar mais as requisições nos demais servidores.
 
Alternativa correta
Porque o servidor em questão pode ser mais fraco, com menos recursos.
 
Alternativa correta! Podemos ter um servidor com menos recursos, que responde mais lentamente, como backup, já que ele apenas será utilizado em casos extremos.

@@08
Faça como eu fiz

Chegou a hora de você seguir todos os passos realizados por mim durante esta aula. Caso já tenha feito, excelente. Se ainda não, é importante que você execute o que foi visto nos vídeos para poder continuar com a próxima aula.

Continue com os seus estudos, e se houver dúvidas, não hesite em recorrer ao nosso fórum!

@@09
O que aprendemos?

Nesta aula, aprendemos:
Aprendemos a dar pesos diferentes a cada servidor no load balancer
Conhecemos os algoritmos de load balancing como round-robin
Vimos como configurar servidores de backup

#### 25/08/2023

@01-FastCGI

@@01
CGI vs FastCGI

[00:00] Boas-vindas de volta a mais um capítulo deste treinamento onde nós estamos conhecendo um pouco desse mundo de servidores web usando nginx. Nesse capítulo o conteúdo vai ser um pouco diferente do que estudamos até agora, porque vamos primeiro estudar algumas bases bastante interessantes, por exemplo, como a web costumava ser e como isso evoluiu de formas diferentes.
[00:22] Tudo vai girar em torno dessa imagem, não se canse do que vou falar, porque é um conteúdo bem interessante, e logo colocamos a mão na massa. Quando a internet surgiu a principal forma de se executar lógica do lado dos servidores, através do que conhecemos como CGI. Lá no início da internet, a internet nasceu, só tínhamos conteúdos estáticos.

[00:45] Tínhamos HTML, que era um portal de informação, você colocava uma receita de bolo, estava lá, na internet. A internet servia para fornecedor documentos através da rede. Como o tempo foi passando as pessoas foram evoluindo seu uso, ao ponto de do lado do servidor podermos usar alguma lógica.

[01:06] Por exemplo, eu poderia contar o número de pessoas que acessaram meu documento. Para isso preciso no meu servidor armazenar alguma informação. Assim a web foi evoluindo para ter lógica sendo executada do lado do servidor, e nos primórdios isso era feito através de CGI, que é common gateway interface, uma interface padrão de um portão de entrada para a web, vamos dizer assim.

[01:30] Como isso funcionava? Tenho meu usuário nos primórdios, ele acessava a internet, e com isso nós chegamos até um servidor web, por exemplo, hoje em dia o nginx, na época acredito que ele nem existia. Mas beleza, chegou no nginx, mas ele precisa de uma lógica, algo precisa ser executado. Mas o nginx é somente um servidor web, ele não executa lógica de programação.

[01:56] Então escrevemos nossa lógica, por exemplo, na época era muito comum fazer isso em C, escrevemos nossa lógica em C e existe uma interface entre um servidor web que entende PHP, e esse programa em C, que entende entrada e saída, seja lá de onde venha, do terminal, de arquivos, etc.

[02:15] Esse CGI, esse protocolo, essa forma de se comunicar permitia que um servidor web, vamos dizer o Apache, por exemplo, chamasse um executável em C, esse executável em C rodava alguma lógica, devolvia uma resposta formatada em html, por exemplo, e o servidor web pegava essa resposta e devolvia para o cliente. Assim funcionava o backend antigamente.

[02:41] Mas isso tem alguns problemas. Por exemplo, sempre que recebo uma requisição tenho que criar um novo processo, esse processo executa inteiro, depois morre e eu devolvo. Essa parte de criar um novo processo a cada nova requisição era muito custosa. Era algo que tornava a performance muito ruim, a performance era péssima com esse cenário.

[03:06] Conforme as necessidades da web foram evoluindo surgiu outro protocolo em cima desse conhecido como fast CGI. Que é basicamente a mesma ideia, mas com o fast CGI você não precisa criar um processo a cada requisição. Chegou uma requisição, imagina nesse mesmo cenário da lógica escrita em C, chegou uma requisição, algum gerenciador de fast CGI vai pegar esse processo, criar e mandar uma mensagem dizendo execute isso aqui.

[03:40] Ele pega de volta e devolve a resposta para o servidor web. Chegou uma nova requisição, esse processo já está vivo, ele só manda outra mensagem, pega de volta e depois manda a resposta. Super simplificando, esse é o conceito por trás do fast CGI.

[03:55] Onde isso é utilizado? Por exemplo, quando falamos de PHP, esse é o formato mais utilizado por aplicações PHP, através de algo que é conhecido como PHP fpn, que é fast CGI e process manager, então gerenciador de fast CGI.

[04:12] Python consegue fazer dessa forma, Java consegue fazer dessa forma, embora não seja comum, linguagens interpretadas via de regra têm uma implementação bastante comum desse protocolo. Mas conforme a web foi evoluindo existem outras formas de responder requisição.

[04:30] Por exemplo, com PHP você tem um servidor escrito no próprio PHP, você pode utilizar ferramentas que mantém o PHP rodando e recebendo requisições. Em Java, por exemplo, esse é o padrão, você tem algum server de container, algum servidor que fica rodando em Java e devolvendo as requisições.

[04:54] Então essa outra abordagem é muito utilizada hoje quando temos muitas operações intensas de IO, de entrada e saída, quando temos muitas conexões com o banco muito pesadas, muitos arquivos muito pesados, muitas requisições para outros locais. Nesse cenário essa ideia de ter um servidor escrito na própria linguagem autocontido, e o nosso nginx só manda essa requisição http para esse servidor autocontido, isso é algo muito comum hoje, mas para a maioria dos cenários mais comuns, vamos dizer assim, um cenário de uma aplicação de blog. Essa aplicação tem duas, três queries SQL em uma requisição, ela pega algo e devolve logo.

[05:35] Para esses cenários fast CGI é uma ótima alternativa. Por isso inclusive é a padrão utilizada por PHP. Porque qual a vantagem dessa abordagem? Esse nosso gerenciador do fast CGI vai enviar mensagem, por exemplo, para o PHP, o PHP devolve a resposta. Aquele processo é limpo, ou seja, as conexões com banco de dados são automaticamente fechadas, os recursos são liberados, qualquer arquivo que tiver sido aberto é fechado, dessa forma não temos desperdício de recurso e nós desenvolvedores não precisamos gerenciar recursos.

[06:11] Nós não precisamos de um pool de conexões com o banco de dados para garantir que não estoura o limite e reutiliza a mesma conexão. Nada disso é preciso, porque o fast CGI está no meio cuidando disso tudo, limpando os processos de tempos em tempos, etc.

[06:25] Agora, naquele cenário onde temos um milhão de requisições por seguinte, queries muito demoradas, várias requisições para outros serviços, ou seja, em cenários mais complexos adotados essa outra abordagem. De novo, cada abordagem tem soluções para esses dois cenários. Como fast CGI é um protocolo, uma forma de se comunicar, ele não é específico de linguagem.

[06:48] A maioria, ou todas as linguagens têm alguma implementação de fast CGI, e já esse cenário de um servidor autocontido, qualquer linguagem que permita abertura de um socket consegue fazer isso, basta que alguém desenvolva esse servidor. E todas as linguagens famosas de hoje em dia possuem servidores assim.

[07:06] Vamos falar desse cenário mais comum, vamos dizer assim, de fast CGI, porque no caso de um servidor autocontido http já fizemos um proxy reverso, temos tudo que precisamos saber para realizar as requisições para o nosso servidor de aplicação. Agora, em um cenário em que temos o fast CGI, como podemos utilizar o nginx?

[07:28] Eu sei que falei muito e essa parte teórica pode ser um pouco chata, mas é muito importante. Agora vamos configurar um serviço usando o fast CGI e vamos fazer o nginx mandar requisições através de sockets usando o protocolo fast CGI.

@@02
Diferença

Vimos neste vídeo como a Web funcionava na época do CGI e como funciona agora com servidores auto-contidos ou através do FastCGI. Um servidor auto-contido é basicamente um servidor web escrito na própria linguagem de programação.
Qual a principal diferença entre CGI e FastCGI?

Alternativa correta
No CGI o processo permanece vivo após o encerramento de uma requisição.
 
Alternativa correta
No FastCGI o processo permanece vivo após o encerramento de uma requisição.
 
Alternativa correta! Ao iniciar um processo FastCGI, ele ouvirá novas conexões e não mais morrerá, ou seja, o mesmo processo continua gerenciando os recursos da aplicação. Usando CGI, a cada requisição um processo é criado e depois morre.
Alternativa correta
Não há nenhuma diferença crucial. São 2 nomes para o mesmo protocolo.

@@03
Configurando o proxy

[00:00] Boas-vindas de volta. Agora que já entendemos o conceito por trás de CGI e fast CGI, quero me conectar através do nginx com algum serviço de fast CGI. Então o que vou fazer? Vou subir um container do Docker que tenha um serviço de fast CGI rodando. Vou usar o PHP fpm, porque ele é muito tranquilo, já vem meio que por padrão configurado, não preciso fazer nada, isso vai facilitar um pouco a vida de quem não conhece nenhum serviço de fast CGI.
[00:35] Se você programa em alguma linguagem que você está habituado a configurar algum servidor, algum serviço de fast CGI pode utilizar ele que tudo aqui deve funcionar exatamente igual. Eu vou utilizar esse serviço, essa imagem do Docker para facilitar a vida da maioria.

[00:50] Como vamos fazer? Em outra aba do meu terminal rodei dois comandos, criei através do echo, que simplesmente exibe uma coisa no terminal, exibi esse conteúdo, que é simplesmente abrir uma tag do PHP e chamar uma função que exibe informações do PHP, e estou redirecionando, ou seja, estou mandando essa mensagem que exibi para o arquivo “index.php”.

[01:15] No final das contas escrevi isso em um arquivo "index.php", em qualquer pasta, e fiz esse comando, estou rodando um container do Docker e logo depois que terminar vou remover ele. Só estou descartando esse container.

[01:33] Estou anexando esse terminal para que tudo funcione como esperamos no terminal e estou liberando a porta 9000 porque é a porta padrão do PHP fpm. E estou dizendo, criando um volume dizendo que tudo que está nessa pasta, e se você estiver no Windows essa parte não vai funcionar, então você coloca o caminho completo da pasta atual, essa pasta vai ser mapeada para a pasta no meu container, minha imagem do Docker.

[02:02] Esse container que estou criando agora vai ser da imagem PHP fpm. O comando padrão dessa imagem já é subir o PHP fpm, já deixa essa aplicação rodando. Ele vai exibir essas duas mensagens, as duas outras foi porque eu fiz alguns testes para garantir que estava funcionando, mas beleza.

[02:22] Tenho aqui uma aplicação que recebe requisições do tipo fast CGI, ela não recebe uma requisição http, ela recebe uma requisição fast CGI. Então consigo acessar esse container, como liberei a porta, através de localhost:9000, sem problema.

[02:41] O que quero fazer agora? Quero configurar um novo serviço, por exemplo, deixa eu reiniciar para garantir que está tudo certo, quero criar um serviço para acessar em localhost:8004, vamos criar um novo serviço. Em /usr/local/etc/nginx/servers/fpm.conf, aqui vou criar um novo servidor, como já estamos habituados, então é bom para praticar.

[03:15] Ele vai ouvir a porta 8004. Nem vou colocar um server name aqui, mas um detalhe importante. Se quero executar arquivos PHP que estão aqui nessa pasta caminho/projeto, vou precisar informar isso a partir do nginx, não vou informar nada por enquanto, vocês vão ver o problema que vai acontecer.

[03:40] Vou colocar nosso location, ou seja, qualquer requisição que chegar aqui vai entrar nesse location, e ao invés de fazer um proxy pass, o que quero fazer um fast CGI pass, e vou passar essa requisição usando um fast CGI, usando esse protocolo fast CGI para o localhost:9000.

[04:05] Teoricamente isso é tudo que preciso, ainda não, mas quase tudo que preciso para as minhas requisições serem passadas do nginx lá para o meu PHP fpm naquele container. Vamos salvar isso, e vou reiniciar o nginx.

[04:22] Vamos ver se já tenho uma resposta diferente. Perfeito, tenho uma resposta diferente, inclusive quando vejo recebi uma requisição, mas repara que nas requisições que eu tinha feito de teste anteriormente tenho algumas informações, como qual foi o verbo http utilizado, qual foi o caminho que chegou essa requisição.

[04:44] Já aqui não tenho informação nenhuma dessa requisição, então o que o PHP fpm fez? Ele recebeu uma requisição, não tem informação nenhuma, então ele só devolveu. Por que não tem informação nenhuma? E se eu tentasse acessar o "index.php"? Repare que ainda assim não tem nenhuma informação chegando, porque o que está acontecendo?

[05:10] Quando fazemos o proxy pass, a requisição HTTP que chegou para nós é reenviada para lá. Podemos fazer modificações como já vimos no treinamento anterior, mas via de regra, o que chegou é enviado. Mas, nesse caso, nós não estamos fazendo uma requisição http de novo, nós recebemos a requisição http no nginx e faz um fast CGI pass, é outro protocolo que está acontecendo aqui.

[05:35] Essa comunicação está sendo feita utilizando não http, mas sim um fast CGI, é outra forma de comunicação, uma comunicação completamente diferente. Então preciso informar, por exemplo, qual verbo http, o caminho que foi requisitado, onde estão os arquivos que ele vai encontrar. Preciso enviar parâmetros para o meu fast CGI.

[06:02] E começamos a pensar, poxa, então vou ter que configurar muita coisa, verbo http, o arquivo que foi requisitado, onde esse arquivo está, todos os cabeçalhos HTTP. Não vale a pena, né? Mas calma que o nginx traz uma grande facilidade para nós. Mas antes de aprender essa facilidade temos que aprender a fazer na unha, né? Mas para aprender a fazer na unha ia levar um bom tempo. Então o que vou fazer? No próximo vídeo vou configurar na prática e vamos entender como essa configuração é feita por baixo dos panos.

@@04
Para saber mais: Unix socket

Neste vídeo nós simulamos o servidor web e de aplicação em máquinas diferentes, mas no cenário onde ambos estão na mesma máquina, a comunicação pode ser feita através de Unix Sockets, ou seja, sem transmissão de dados pela rede.
Isso pode trazer um pequeno ganho de performance, então vale a pena o estudo.

@@05
Parâmetros adicionais

[00:00] Boas-vindas de volta. Já temos uma requisição HTTP chegando e outs usando o protocolo fast CGI indo para o nosso PHP fpm. Mas ela está incompleta, então o que vou fazer? Repare que vai ser bastante simples começar a correção. Vou adicionar um include fastcgi.conf, vou salvar e recarregar o nginx.
[00:25] Teoricamente algumas coisas já vão mudar aqui. Repare que quando atualizo tenho um file not found, já começou, um erro diferente já é um avanço. Mas vamos ver. Repare que a requisição já chegou corretamente. Já sabemos que é uma requisição usando verbo get para o arquivo "index.php".

[00:44] Perfeito, a primeira etapa está ótima, mas agora o nosso PHP fpm não sabe onde buscar esse arquivo "index.php", por quê? Vamos lá. Tenho essa configuração e incluí, fiz o include do arquivo “fastcgi.conf”. Vamos ver o que tem nele.

[01:10] Nesse arquivo temos várias diretivas de fast CGI param, basicamente o que isso faz é adicionar um parâmetro a essa nossa chamada fast CGI. Então aqui é o parâmetro que estou adicionando e o valor dele. Repare que ele está mandando qual o arquivo, isso chegou certo, a query string se tivesse, no nosso caso não tem, qual o método que foi utilizado nesse request, content type, ele manda tudo que nós precisamos.

[01:38] Só que tem um detalhe que ele está mandando document root, e nós não informamos qual é esse document root, então o PHP fpm não está sabendo onde encontrar.

[01:50] Eu vou adicionar nesse nosso root que é naquele caminho que eu configurei, que eu tenho nosso arquivo na máquina do PHP fpm, que é o /caminho/projeto, então na máquina que tem o PHP fpm, a pasta raiz do nosso projeto é /caminho/projeto, então vou informar isso aqui.

[02:15] Com isso, quando faço o reload teoricamente tenho o nosso arquivo de PHP sendo executado. Toda a nossa lógica pode ser executada lá pelo servidor do PHP fpm, ele devolve isso para o nosso nginx e o nginx manda para o cliente.

[02:30] O que aconteceu aqui? Vamos lá. Quando mando uma requisição para esse serviço, para o serviço de 8004, em qualquer url, o que ele está fazendo? Está caindo no location que inclui o arquivo de configurações que o próprio nginx traz para nós, que tem todos os parâmetros necessários para o fast CGI, então agora com todos os parâmetros inclusos, estou fazendo como se fosse um proxy reverso, mas não estou mandando outra requisição http, estou mandando uma requisição utilizando um fast CGI.

[03:05] Existe uma comunicação por rede, mas não usando http. É um protocolo mais enxuto que passa informações em um formato um pouco mais comprimidas, e dessa forma nosso PHP fpm não precisa receber toda a requisição http no formato http, além disso, por ser um fast CGI, ou seja, por utilizar aquele protocolo, e aquela especificação, esse processo que ele fica rodando não morre e quando ele recebe uma requisição do tipo fast CGI ele manda uma mensagem, executa o PHP e devolve a resposta.

[03:46] Ele limpa aquele processo do PHP que não é o processo completo que precisa ficar ouvindo a requisição, é algo mais limpo, vamos dizer assim, que tem uma performance melhor do que um tipo CGI, mas ainda conseguimos ter essa limpeza.

[04:02] Recebi uma requisição http, mandei essa comunicação por rede usando o protocolo fast CGI e nosso gerenciador de fast CGI, esse gerenciador de processos recebe e faz o que tem que fazer. De novo, estou usando PHP fpm por ter um container do docker bem simples já configurado. Você pode utilizar algum gerenciador de processos do Python, do Hub, do Java, de C#, de qualquer linguagem, na verdade, de C, C++, linguagens compiladas.

[04:35] Se existe algum gerenciador de processos fast CGI para sua linguagem que execute na sua linguagem você vai conseguir fazer exatamente a mesma configuração. Então beleza, dessa forma temos outra possibilidade aqui. Posso ter, por exemplo, um proxy reverso para pegar nossos arquivos estáticos usando load balancer e tudo mais, e posso ter o fast CGI pass rodando para os dados, mandando essa requisição para um servidor de aplicação que usa fast CGI.

[05:05] Repare que as possibilidades começam a aumentar. Posso ter um load balancer na frente e esse load balancer manda para um proxy reverso, esse proxy reverso pode mandar requisições http para outros serviços ou já devolver os arquivos estáticos, e pode mandar esse fast CGI para um servidor de aplicação em outro local. E nós podemos ir aumentando nossa arquitetura do sistema, nosso design system aqui com vários serviços diferentes.

[05:35] E obviamente, de novo, falei bastante isso no treinamento anterior, mas é bom recapitular aqui, cada um desses serviços que crio poderiam facilmente estar em servidores diferentes, em máquinas diferentes, por isso o nome da diretiva é server, normalmente está em um servidor diferente, então um servidor seria o load balancer, o outro teria um proxy reverso, o outro faria esse fast CGI pass e assim por diante.

[06:02] Falei bastante sobre fast CGI, vamos de novo voltar um pouco para o mundo só de servidores web, sem servidores de aplicação e aprender coisas novas, como, por exemplo, manipular outras coisas bastante interessantes do próprio HTPP.

@@06
Faça como eu fiz

Chegou a hora de você seguir todos os passos realizados por mim durante esta aula. Caso já tenha feito, excelente. Se ainda não, é importante que você execute o que foi visto nos vídeos para poder continuar com a próxima aula.

Continue com os seus estudos, e se houver dúvidas, não hesite em recorrer ao nosso fórum!

@@07
O que aprendemos?

Nesta aula, aprendemos:
Entendemos como funcionava a Web nos primórdios
Aprendemos sobre o conceito de CGI e as vantagens do FastCGI
Vimos como enviar as requisições para um servidor FastCGI
Aprendemos a manipular os parâmetros FastCGI

#### 26/08/2023

@03-Performance

@@01
Cache HTTP

[00:00] Boas-vindas de volta a mais um capítulo deste treinamento, onde estamos aprofundando nossos conhecimentos com esse servidor web nginx. Já mergulhamos um pouco mais na parte de load balancers, já vimos sobre fast CGI. Vamos voltar para o mundo http e aprender sobre um assunto que é muito importante não só para nós que estamos configurando um servidor web, mas também para quem é desenvolvedor web, para quem é desenvolvedor backend, frontend.
[00:28] Esse assunto é performance. Vamos falar sobre performance, vamos falar sobre como o nginx pode nos ajudar nessa parte de performance. Acho válido citar que tem dois cursos fenomenais sobre performance web aqui na plataforma, inclusive tem um guia de estudos do Sérgio Lopes que é fera nessa área, que tem além desses dois cursos dele também alguns outros recursos. Vale muito conferir.

[00:52] Mas como estamos estudando nginx, acho que não dá para deixar isso passar em branco, então vamos dar uma estudada sobre isso. Para começar, o que tenho aqui? Criei uma nova pasta performance e nessa pasta tenho uma imagem de um meme bobinho e um arquivo, um HTML.

[01:11] Você pode pausar o vídeo e escrever esse HTML, ou você pode copiar ele da transcrição, a imagem você pode usar literalmente qualquer imagem, não tem nada de mais no que vamos fazer específico desses arquivos. Mas vamos lá, vamos configurar um servidor para servir esse HTML que exibe essa imagem.

[01:30] Vou criar um usr/local/etc/nginx/servers/performance.conf, aqui vou criar um novo servidor e vou vir 8005, acho que essa porta está disponível, nosso root, ou seja, o caminho onde o nginx vai buscar nossas pastas, nossos arquivos, é esse, e nosso index é o arquivo “index.html”.

[02:02] Repare que não coloquei nenhum location, porque essas configurações são para o servidor todo, e não para uma parte específica dele, então não tem problema nenhum deixar fora do location. Vamos fazer um reload para recarregar o servidor, para o servidor recarregar as configurações, e vamos acessar esse 8005.

[02:22] Temos nosso feliz aniversário “bobinho”, e agora vamos para o ponto de performance, onde quero chegar. Quando faço uma requisição, então quando digito esse endereço e dou enter, ou quando atualizo, quando aperto f5, o que acontece? Uma requisição é feita para esse endereço, isso cai no nosso arquivo, ele vê que tem que pegar esse index.html.

[02:44] Esse index.html é devolvido para o navegador, o navegador recebe isso e faz um parse disso, faz a interpretação desse arquivo e vê que precisa buscar essa imagem, então ele volta no servidor, pega essa imagem e depois exibe lá. Então temos duas requisições nesse caso, mas o que acontece?

[03:10] Essa imagem de uma requisição para outra, as chances dela ter mudado são pequenas, é baixa a chance dessa imagem ter sido alterada. A página, em si, é mais comum que alteremos, façamos correções, mude o estilo, etc., mas a imagem vai ser sempre a mesma.

[03:25] Então, o que quero fazer? Quero dizer, olha só, navegador, quando você baixar essa imagem pode manter ela salva aí, não precisa me pedir não, pode usar a que você tem salva. Esse é o conceito de cache, e muitos navegadores já fazem cache por padrão, mesmo que o servidor não instrua ele a fazer isso.

[03:44] Por exemplo, estou usando um navegador que é bastante parecido com o Chrome, então podemos chamar ele de Chrome aqui, e via de regra o Chrome faz algum cache, mesmo que o servidor não instrua, principalmente quando estamos em localhost. Ele assume algumas coisas a mais, ele faz algumas assunções, ele parte do princípio que ele pode fazer algumas verificações a mais, etc.

[04:14] Então, o que quero analisar? Vou clicar com o botão direito em qualquer lugar da página e vir em inspecionar. Aqui temos algumas abas, dependendo do navegador pode mudar um pouco, mas é bem semelhante, venho em network.

[04:26] Repare que tenho um disable cache, se marco isso e faço uma requisição ou se venho aqui e atualizo, ele nunca vai cachear. Repare que ele sempre traz o conteúdo completo, nunca faz cache, mas se eu não marco a opção de desabilitar cache e atualizo, repara que o Google Chrome já faz um cache, mantém essa imagem e não vai ao servidor buscar.

[04:52] Então quando pego o tamanho dessa imagem, quando tento ver o tamanho dessa imagem ele mostra que nada precisou ser transferido pela rede, esse arquivo já estava em memória e ele conseguiu servir.

[05:05] Então o que quero fazer é a partir do meu servidor instruir para o Google Chrome ou para qualquer outro navegador, qualquer cliente. Olha só, quero que você realize o cache, porque aqui o navegador está fazendo por conta própria e ele pode fazer o cache ou não, não sei quanto tempo ele está armazenando isso em cache, pode ser dez minutos, dez segundos, um dia, para sempre, não sei.

[05:26] Então o que quero fazer? A partir do nginx, ou seja, a partir do meu servidor web, quero informar quem fez a requisição para essa imagem, você pode armazenar isso, essa imagem expira daqui trinta dias, então você pode mandar ela em trinta dias, se você precisar dela não precisa mandar requisição, ela está válida.

[05:46] Trinta dias depois eu recomendo que você faça a requisição de novo. Então como podemos fazer isso? Como podemos informar algo para o navegador? Podemos utilizar cabeçalhos http. Podemos enviar cabeçalhos http para passar essa informação para o nosso navegador. Repare que tenho tanto os cabeçalhos da requisição quanto da resposta.

[06:11] Então quero que na resposta, ou seja, o que vem do servidor, adicione alguma informação de cache. Quero informar que essa imagem expira em trinta dias. Vamos lá para a nossa configuração. Estou na configuração, sempre que eu acessar um location que termine com .jpg, falamos um pouco de expressão regular no treinamento anterior, mas deixa eu refrescar a memória.

[06:40] Quando tenho esse til o que vem depois é uma expressão regular, uma Regex. Esse contra barra significa que o ponto que vem depois é realmente um ponto e não qualquer caractere, porque um ponto numa expressão regular pode ser qualquer caractere, e depois jpg e o cifrão indica que é o final, ou seja, esse location vai casar, vai bater, servir para qualquer arquivo que termine com .jpg, para qualquer localização, URL que termine com .jpg.

[07:11] Então o que quero fazer aqui é informar que qualquer informação, qualquer recurso, no caso aqui imagem, que for acessado usando esse location vai expirar em trinta dias. Dessa forma sempre que alguma regra cair, alguma requisição cair nessa regra de location, nós vamos adicionar essa informação na resposta, informando que isso expira em trinta dias.

[07:40] Então vamos salvar, recarregar, e agora vamos fazer uma nova requisição aqui e quando clico vamos conferir se temos os detalhes. Lembra que o Chrome já fez um cache mesmo sem pedirmos, então ele não mandou requisição. Vou desabilitar o cache para que o Chrome faça a requisição, e quando eu atualizo agora o que vamos ter aqui?

[08:05] Repare que tenho uma informação de cache control, vou desabilitar essa opção. O que o cache control quer dizer? Basicamente está dizendo que esse recurso tem a idade máxima, ou seja, pode durar esse número de segundos ou minutos, eu acho que é segundos. Essa quantidade de tempo, e isso quer dizer trinta dias.

[08:25] O nginx já fez essa conta e mandou esse cabeçalho para nós, tudo que precisamos era informar que isso expira em trinta dias. Mas ele também manda outro cabeçalho, o expires, então como estou gravando isso no dia 17 de abril, ele mandou que isso vai expirar no dia 17 de maio.

[08:45] Ele já passou essas duas informações, porque de novo, super resumindo esse conteúdo, os navegadores antigamente, alguns navegadores interpretavam melhor esse cabeçalho, outros davam mais importância para esse cabeçalho, então a recomendação sempre foi enviar os dois.

[09:05] Hoje em dia os navegadores modernos sabem lidar com os dois, então qualquer um dos dois cabeçalhos vai servir, mas ainda é recomendado enviar os dois porque alguns outros clientes que não são navegadores podem por algum motivo depender de algum desses cabeçalhos, então o ideal é mandar os dois para garantir.

[09:20] Dessa forma agora quando vamos tentar acessar essa página de novo o conteúdo não precisa ser transferido pela rede, ou seja, ele vai ser buscado, vai ser servido através do próprio cache, ou seja, isso já está salvo no navegador.

[09:35] Agora um último detalhe sobre cache é que o cache control tem algumas coisas bastante interessantes, além de max age, temos alguns outros valores possíveis. Por exemplo, um caso onde podemos adicionar outro valor. O cache pode ser armazenado não só pelo navegado. No nosso caso o navegador está armazenando, mas entre um servidor que possui o arquivo e o navegador do cliente final podem existir diversas etapas.

[10:02] Por exemplo, um proxy reverso, um cdn, que é content delivery network, pode ter um proxy na própria rede. Se eu quiser informar posso informar que qualquer uma dessas etapas entre servidor e cliente pode realizar o cache também. Então posso fazer isso através de um cabeçalho cache control com valor public, e caso você não saiba um cabeçalho http pode ter vários valores, posso enviar ele várias vezes.

[10:33] Vou adicionar um cache control public, vamos lá no nosso performance e no nosso location vou adicionar um cabeçalho, então add_header Cache-Control public, vou salvar, fazer o reload e quando venho aqui vou desabilitar o cache para ele ir ao servidor receber essa informação, e atualizo.

[10:55] Posso desmarcar isso e quando dou uma olhada tenho cache control public além do cache control com max age. Ou seja, se eu tiver algum cdn, algum servidor de cache específico, um proxy reverso, um proxy na própria rede, tudo isso vai ver isso e caso esse intermediário saiba realizar cache ele vai ter essa informação e vai ver que pode fazer cache, e vai usar essa informação.

[11:22] Então aqui temos um resumão sobre cache HTTP, de novo, tem um curso específico sobre performance onde muito mais detalhes sobre caches são explicados, mas eu acho que isso é um bom início. Só que isso não é tudo que temos a falar sobre performance, então continuamos no próximo vídeo.

@@02
Papel de cada um

Vimos neste vídeo como enviar um cabeçalho HTTP além de informar a expiração do cache de alguns recursos.
Sobre cache, marque a alternativa correta:

Cache HTTP é um recurso exclusivo do Nginx.
 
Alternativa errada! Nginx apenas facilita o envio dos cabeçalhos HTTP, mas esse conceito é da Web em geral.
Alternativa correta
O navegador pode ignorar os cabeçalhos de cache e não cachear o recurso.
 
Alternativa correta! Tanto isso é verdade que temos a opção de desabilitar o cache do nosso navegador. Os cabeçalhos de cache instruem o navegador sobre quanto tempo ele pode/deve manter o recurso em cache, mas cabe a ele aceitar essa instrução ou não. Todos os navegadores modernos tendem a seguir a instrução a menos que configuremos de forma diferente.
Alternativa correta
O cache de um recurso sempre é buscado do disco.

@@03
Compressão

[00:00] Boas-vindas de volta. Estamos brincando um pouco, falando sobre performance e como o nginx nos ajuda com isso. Falamos sobre cache, mas nem só de cache vive a performance de uma aplicação web, então vamos entender o que mais o nginx faz para nós de forma super simplificada que pode ajudar bastante na performance.
[00:20] Você já deve ter notado que a fonte mudou e o que eu fiz? Acessei getbootstrap.com, vim em download e baixei esse arquivo zip, desse zip peguei o arquivo bootstrap.min.css, e esse arquivo fiz o link no nosso html para que o html utilize esse CSS.

[00:42] A partir de agora temos mais um arquivo sendo incluído aqui, ou seja, quando atualizo tenho um número maior de conteúdo sendo transferido, tenho aproximadamente 170kbps sendo transferidos aqui. Isso é bastante pouco porque é um exemplo bem pequeno. Em um cenário real você pode ter megas sendo transferidos.

[01:02] Isso é baixado da internet, pode ser baixado pela internet de um celular, por exemplo, um 3g, em alguns lugares do Brasil um 2g infelizmente. Então essa conexão, essa transferência pode ser lenta. Quanto menor a transferência melhor. Óbvio que existem otimizações que podemos fazer na imagem, por exemplo, mas isso foge do escopo deste treinamento. Lá no treinamento de performance isso é citado.

[01:30] O que podemos fazer com o nginx para que esse número diminua, de 170kbps para alguma coisa menor do que isso? Qualquer melhor já pode ajudar. Imagine que você quer me enviar um arquivo por e-mail, mas esse arquivo é muito grande, o que você faz para melhorar um pouco a experiência dessa transferência? Você comprime esse arquivo usando gzip, zip ou rar, tar, alguma coisa assim, algum algoritmo de compressão.

[02:00] Na web é muito comum enviarmos arquivos comprimidos. Então o que podemos fazer? Podemos vir no nginx e informar que nesse servidor quero ativar o gzip, quero ativar a compressão no meu servidor. O nginx já tem por padrão um módulo de compressão instalado, então quando faço isso habilito esse módulo no meu servidor.

[02:33] Deixa eu atualizar e recarregar, e vamos ver o que acontece quando atualizo agora. Repare que estou desabilitando o cache para vermos o tamanho de toda a transferência mesmo. Quando faço isso tenho os mesmos 170kbps.

[02:48] Por padrão o nginx vai comprimir somente o HTML que estamos enviando. Se eu quiser comprimir algum outro tipo de arquivo preciso informar para ele, porque o nginx não sabe o que faz sentido comprimir, o que não faz no nosso cenário. Talvez nós já estejamos fazendo compressão de alguma outra coisa de outra forma, então vamos informar para ele.

[03:08] Mas se você reparar em um detalhe, já tenho do content encoding, ou seja, esse detalhe meu HTML já está comprimido, mas ele é tão pequeno que nem faz diferença, só adicionamos um cabeçalho a mais, então talvez tenha ficado até um pouco maior.

[03:25] Agora vamos ver se isso vai fazer diferença no nosso CSS, que tem 155kbps, é quase toda a nossa transferência. Vou informar os gzip types, ou seja, os tipos que quero fazer compressão. E aqui posso ter image.jpg, mas esse tipo de compressão funciona muito bem com arquivos de texto, arquivos de imagem, arquivos binários não costumam ser muito beneficiados, mas vamos dar uma olhada para ver como isso fica.

[03:55] Também posso adicionar o text/css, vamos ver como fica essa compressão. Vou recarregar e vamos atualizar para ver. Tenho 155 e 14.3. Quando atualizo repare que a imagem nem foi muito otimizada, então nem vale a pena. Acabo dando mais trabalho para o servidor para tentar comprimir a imagem e ele nem consegue fazer muita coisa.

[04:20] Agora, aqui temos um ganho absurdo. De 155kbps para 30kbps. Isso é um ganho muito grande, imagine em um site inteiro. Aqui foi um arquivo só, mas imagine no site real, com vários arquivos de CSS, Java script, cvg e qualquer outra coisa que contenha texto.

[04:40] Com essa simples adição de duas linhas, inclusive deixa eu tirar da imagem para não ficarmos gastando processamento do servidor à toa, com essas duas linhas temos um ganho absurdo na nossa performance. Repare que daqueles 170kbps temos agora 45kbps sendo transferidos. Isso é muito útil principalmente na nossa realidade brasileira, onde temos uma banda bastante estreita, uma internet bastante ruim em vários lugares do Brasil.

[05:12] Falamos sobre cache para evitar requisições, falamos sobre compressão para fazer com que essas requisições transfiram menos dados, agora vamos falar de outro detalhe que está já sendo feito aqui, mas que precisamos conhecer quando falamos de performance, eu pelo menos acho muito válido ter esse conhecimento. Vamos falar um pouco mais para finalizar sobre conexões em si.

@@04
O que comprimir

Vimos neste vídeo como a compressão pode ajudar a diminuir a quantidade de recursos trafegados pela rede, e como é fácil realizar compressões com Nginx.
Que tipo de recursos são os mais beneficiados pela compressão com gzip na web?

Arquivos de texto (html, css, js, svg, etc).
 
Alternativa correta! Arquivos de texto podem facilmente ser comprimidos e são os que mais levam vantagem desta técnica. Arquivos binários ou arquivos de imagem, por exemplo, naturalmente já são comprimidos, por isso o efeito seria bem menor (ou inexistente).
Alternativa correta
Arquivos binários e de imagem.
 
Alternativa correta
Todos os formatos de arquivo.

@@05
Conexões

[00:00] Boas-vindas de volta. Vamos falar sobre o próximo de conexão entre um cliente e o nginx e como podemos melhorar um pouco isso. Como falamos lá no início do primeiro treinamento, o nginx funciona com um processo principal e vários worker processes, então, o que acontece?
[00:22] Por padrão nossa configuração está com um worker processes, e segundo a própria documentação você colocar um worker processes para cada um dos núcleos que o seu computador, que seu servidor tem é uma boa pedida, assim cada um dos núcleos vai poder receber um worker e esse worker vai poder receber várias requisições.

[00:50] Além disso, podemos definir o limite que cada worker vai ter de conexões. E isso vai depender muito do tipo de requisição que seu sistema recebe, do sistema operacional, o limite de found scripters, etc. Então vamos mudar somente o worker processes aqui.

[01:08] Vamos mudar o limite de um para auto, assim o próprio nginx vai ver quantos núcleos nossa CPU tem, no meu caso são oito, então poderia definir como oito, mas vou deixar essa responsabilidade para o nginx. E isso não está no nosso arquivo de um serviço específico, isso está no “nginx.conf”, então em worker processes vou botar auto.

[01:30] Vou fazer um reload e agora quando o servidor sobe, quando foi nesse reload, ele já cria mais processos, um processo worker para cada um dos núcleos. Agora nosso servidor já está pronto para receber muito mais conexões. De novo, através de testes, benchmark, estudos sobre seu tipo de aplicação, podemos aumentar esse worker connections, às vezes diminuir, isso depende de teste, é muito relativo, então não vamos mexer nele aqui.

[02:00] Outro detalhe muito interessante é algo que o nginx faz sobre keep alive connections. Quando faço a requisição, no localhost vemos os headers de resposta e temos um connection keep alive, temos um cabeçalho que não sei muito bem o que significa, mas aqui tem um link para uma explicação do próprio nginx.

[02:26] Quando damos uma olhada nessa explicação nos deparamos com algumas imagens, quando não temos o keep alive ativado, o que acontece? O nosso cliente, o navegador fez uma requisição? O que acontece? Uma conexão é aberta com nosso servidor, essa primeira parte, nós abrirmos essa conexão, enviamos essa requisição HTTP, esse pedido.

[02:52] O servidor vai fazer o que tem que fazer, devolve, e o cliente lê essa resposta http, faz o parse dela e fecha essa conexão. Então esse processo precisa ser realizado para cada requisição que nós fazemos, mas às vezes como temos mais de uma requisição praticamente ao mesmo tempo, muito próximas, e para o mesmo servidor, para fazer uma página só, então será que preciso mesmo criar três conexões diferentes?

[03:22] Talvez faça sentido criar três conexões diferentes, mas nem sempre isso é útil, então podemos utilizar o conceito de keep alive, onde o navegador vai manter essa conexão aberta por algum tempo. Então o que ele faz? A conexão é aberta, ele manda o pedido, quando ele recebe o pedido ele espera por algum tempo, você pode definir esse tempo, quanto tempo você prefere que ele espere.

[03:46] Se passar desse tempo e nenhuma outra requisição precisa ser feita, ele vai fechar. Caso contrário ele vai utilizar essa mesma conexão para fazer a requisição, ou seja, não precisamos dessa primeira etapa aqui e ganhamos uma performance bem interessante.

[04:04] Para configurar esse keep alive, como você já viu, utilizamos cabeçalhos. Temos um cabeçalho content connection igual o keep alive, e nós temos o cabeçalho keep alive, onde podemos definir o timeout, podemos definir outras opções.

[04:20] Se dermos uma olhada em http keep alive caímos na documentação e temos alguns parâmetros de timeout e max, o número máximo de pedidos que podemos ter sendo respondidos por uma única requisição, e o timeout, ou seja, o tempo que podemos manter essa requisição aberta sem pedidos antes de fechar, e assim podemos enviar esses dados.

[04:45] Então, se quero adicionar um cabeçalho http você já sabe como podemos fazer. Vou copiar esse valor para usar igual e vamos à configuração desse nosso arquivo de performance adicionar um cabeçalho, add_header Keep-Alive *timeout=5, max=100. Vamos recarregar e ver se não digitei nada errado.

[05:11] Quando realizo essas requisições de novo, ignorando o cache, vamos ver se recebemos essa nova resposta do keep alive, e está lá nosso keep alive configurado, dessa forma se dentro de cinco segundos eu tentar realizar alguma requisição dentro dessa página, a mesma conexão vai ser aproveitada.

[05:30] Recapitulando, configuramos quantas requisições, como as requisições vão ser lidadas pelo nosso servidor, aumentando o número de processos. Poderíamos mudar o número de conexões que cada processo recebe e estamos instruindo o navegador a manter as conexões abertas utilizando keep alive.

[05:48] Falamos de performance, mas ainda tem muita coisa que podemos ver, que podemos utilizar do nginx além dessa parte específica de http, inclusive essa parte de cache que vimos no primeiro vídeo desse capítulo pode ser feito sem ajuda do navegador. Nós podemos ter cache do lado do servidor. Então, o que acha de conversarmos sobre esse assunto no próximo capítulo?

@@06
Para saber mais: Keepalive

Vimos neste vídeo como configurar o cabeçalho Keep-Alive, mas se você quiser mais detalhes sobre os possíveis valores, pode conferir esta página: Keep-Alive.

https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/Keep-Alive

@@07
Faça como eu fiz

Chegou a hora de você seguir todos os passos realizados por mim durante esta aula. Caso já tenha feito, excelente. Se ainda não, é importante que você execute o que foi visto nos vídeos para poder continuar com a próxima aula.

Continue com os seus estudos, e se houver dúvidas, não hesite em recorrer ao nosso fórum!

@@08
O que aprendemos?

Nesta aula, aprendemos:
Aprendemos sobre cache HTTP
Relembramos como enviar cabeçalhos HTTP
Conhecemos e habilitamos a compressão com gzip
Aprendemos a configurar o número de processos do Nginx
Vimos as vantagens de manter uma conexão HTTP aberta

#### 27/08/2023

@04-Cache

@@01
Caminho de cache

[00:00] Boas-vindas de volta a mais um capítulo deste treinamento, onde estamos brincando com um servidor nginx, esse servidor web, que além de ser um servidor web também serve como proxy reverso, load balancer e outra coisa que vamos ver agora que é como servidor de cache.
[00:15] Podemos criar inclusive cdns utilizando nginx, mas isso já fugiria do escopo deste treinamento, então vamos focar com calma na parte de cache. Só para recapitular o conceito de cache, vamos pegar esse exemplo do cache HTTP. O navegador fez um cache dessa imagem, nós sabemos, nós configuramos isso.

[00:38] Posso com tranquilidade vir e apagar essa imagem. Repare que estou na pasta mesmo, apaguei a imagem, quando atualizo o navegador continua exibindo ela, porque essa imagem está no cache, não está indo no servidor. Agora se vier e disser que vou desabilitar o cache, quando atualizo não encontra mais.

[00:58] Conceito de cache recapitulado. Agora imagina um cenário onde tenho uma URL que o processamento dela demora. É o mesmo processamento para todo mundo que vai acessar, só que ele demora, é feito por um servidor de aplicação, precisa fazer uma query, alguma coisa do tipo.

[01:15] Imagine o site da Alura. Eu venho, carreguei esse HTML. Independente de ser eu, você ou qualquer outra pessoa acessando, essa página é a mesma, com o mesmo vídeo, com as mesmas categorias, com as mesmas formações aparecendo, com os mesmos patrocinadores, esse mesmo número de cursos. Ou seja, essas informações que são computadas no servidor são exibidas para todo mundo igual.

[01:45] Então eu não preciso que esse cache seja guardado só no navegador, posso armazenar esse cache em um servidor central, no meu load balancer, por exemplo, ou algum proxy reverso que eu tenha, e quando outra pessoa fizer a requisição esse número de cursos já foi computado, essa query para buscar as formações já foi calculada e não preciso ficar indo no meu PHP, no Java, no Python, no Hub para fazer isso tudo de novo.

[02:12] Quero fazer com que meu nginx faça isso, que ele atue como um cache. Por exemplo, quando acessar o meu servidor que está indo no meu fast CGI com php fpm, quero que acessei uma vez, essa minha configuração do PHP, do fpm que está mandando o fast CGI param, quero fazer com que ele armazene o cache de alguma forma, então esse cache precisa ser guardado em algum lugar dessa máquina, desse servidor. E sempre que essa requisição for feita para esse location quero informar que quero buscar aquele cache.

[02:55] Vamos lá que são muitas coisas que precisamos fazer aqui, mas queria falar sobre o conceito primeiro. Como falei, dentro de um location vou poder informar que quero buscar o cache de alguma localização, mas antes disso preciso configurar uma localização de cache, e isso não fica dentro de location, isso não fica dentro de server, fica dentro de http, que é aquele que está no nginx.conf, aquele arquivo padrão.

[03:20] Ou seja, posso colocar algo aqui fora. O que posso informar aqui? Na documentação do nginx, na verdade, é mais um tutorial, ele informa essa diretiva proxy_cache_path, e nós poderíamos utilizar ela sem problema, mas no nosso caso essa configuração de servidor não é um proxy reverso.

[03:42] Se fosse o caso, eu utilizaria o proxy_cache_path, simples. No meu caso o que vou precisar é do fastcgi_cache_path, então vamos utilizar ele para realizar nossa configuração. O que preciso passar de informação? Preciso necessariamente de um caminho, obviamente, vou colocar em /tmp/cache, preciso passar o keys_zone, se não me engano é esse o nome, que vai informar o nome desse cache que vou salvar aqui e o tamanho dele.

[04:18] Posso aumentar, diminuir, depende do seu cenário, eu vou botar 10mb que é um valor que tirei da cabeça. E eu poderia informar algumas outras várias coisas. Por exemplo, posso ter níveis de cache. Se informo dessa forma, 1, 2, por exemplo, /data/nginx/cache, informo que vou ter um cache de dois níveis, então dentro de cache ele vai criar duas pastas. Níveis diferentes.

[04:45] Para separar o cache em vários níveis para o sistema operacional não ter uma pasta com vários arquivos, o que torna um pouco mais lento. Vamos configurar esse segundo nível de cache, esse cache de dois níveis no diretório. Vamos lá, cache de dois níveis aplicado.

[05:05] Posso informar quanto tempo esse dado precisa estar inativo para se tornar inválido, posso informar muita coisa, só que por enquanto para mim isso é suficiente. Mas temos um detalhe que é o fastcgi_key, preciso informar que chave vai ser utilizada para fazer esse cache, posso utilizar a URL, por exemplo, bastante simples.

[05:30] Mas repara que esse fastcgi_cache_key já entra em outro local de location, então vamos dar um passo atrás e entender o que está acontecendo aqui. Dentro do meu servidor, seja não de um serviço específico, ou seja, dentro do meu computador, da minha máquina, que está na diretiva http, vou informar onde quero guardar caches. Posso ter cache para fast CGI, posso ter proxy cache, e é na mesma lógica, poderia ter um proxy cache em outra pasta, ou até na mesma pasta sem problema, também usando dois níveis de cache.

[06:14] E aí obviamente precisaria dar outro nome, isso poderia ser proxy, seria de 1mb só, ou seja, no meu computador, posso ter vários locais de cache onde posso armazenar as coisas. E esses vários locais podem ser compartilhados entre vários servers, ou seja, vários serviços, servidores, e em vários location diferentes.

[06:35] Aqui configuramos a nossa máquina para ter esse espaço de cache. Vou tirar esse meu proxy porque só vou utilizar o fast CGI, e se der uma olhada em tmp cache tenho algumas coisas que estava fazendo nos testes, então vou remover esse tmp cache.

[06:55] Removi essa pasta, se venho em tmp, não tenho mais a pasta cache. Se tento acessar não existe. Quando faço agora um nginx reload, o que vai acontecer? Ele cria essa pasta de cache já para mim. Repare que ele cria ela por enquanto vazia, mas agora essa pasta existe e está pronta para o nginx manipular, fazer o que quiser, adicionar arquivos, ler arquivos, remover arquivos, etc.

[07:25] Nós começamos a dar o primeiro passo para transformar nosso servidor nginx em um servidor de cache. Dado esse primeiro passo no próximo vídeo nós vamos efetivamente fazer esse cache funcionar.

@@02
Cache no servidor

Vimos neste vídeo que é possível transformar nosso servidor em um servidor de cache, armazenando dados de resposta para não reprocessar determinadas requisições.
Em que cenário faz sentido armazenar cache no servidor?

Em nenhum cenário faz sentido realizar cache no servidor. Este papel é do cliente (navegador).
 
Alternativa correta
Em todo o nosso site é recomendado ter tanto cache no servidor quanto no cliente (navegador).
 
Alternativa errada! Nem sempre vamos querer cachear alguma requisição. Informações que dependem do usuário logado ou que mudam constantemente não devem ser cacheadas.
Alternativa correta
Em URLs que precisam de processamento e não mudam de usuário para usuário.
 
Alternativa correta! A página inicial da Alura é um ótimo exemplo. Nós não precisamos realizar todas as queries de cursos, formações, etc o tempo todo. Podemos executar uma vez só e armazenar o html montado em cache.

@@03
Usando o cache

[00:00] Boas-vindas de volta. Já fizemos a primeira etapa, que é configurar o local onde esse cache vai ser armazenado. Agora podemos ir à configuração. E aqui sempre que eu acessar esse location que faz o fast CGI pass posso utilizar aquele cache. Então posso utilizar o fastcgi_cache e vou utilizar aqui o nome da zona que estou utilizando, que é o fpm.
[00:30] Só que aqui quando eu tentar fazer um reload ele vai dar um aviso, que não tenho nenhum fastcgi_cache_key, ou seja, como o nginx vai identificar cada uma das requisições para realizar o cache? Ou seja, todas as requisições para a mesma URL vão entrar nesse cache ou tem que ser usando o mesmo método? O verbo http. E se o host mudar?

[00:55] Podemos adicionar essa informação aqui no fastcgi_cache_key, então vamos nessa fazer isso. Nesse cache_key informamos qual vai ser a chave do cache, ou seja, quais informações precisam bater para chegarmos nesse cache. Precisa ser sempre o mesmo método http para a mesma URL? Então vamos fazer isso. Se você der uma olhada na documentação, temos como exemplo somente o request_uri, podemos fazer somente isso, tudo para essa URL vai cair nesse cache.

[01:28] Mas eu vou incrementar um pouco e adicionar o cache_key como request_method e request_uri. Vamos lá, se não escrevi nada errado vamos ter esse cache sendo implementado agora. Repare que no nosso fpm vou dar enters para separarmos o pré-cache e o pós-cache, vou fazer uma requisição, se tudo der certo e nada der errado, requisição funcionando.

[01:55] Aqui vejo que uma requisição realmente foi feita. Agora quando tento de novo, quando atualizo repare que a resposta foi um pouco mais rápida, não sei se deu para perceber, mas quando venho aqui ainda não está sendo feito o cache. Vamos ver o que aconteceu na configuração, tenho o fastcgi_cache, tenho o fastcgi_cache_key e tenho o cache_path, vamos ver se utilizei o mesmo nome.

[02:20] A princípio está tudo certo, vamos ver o que deu mole aqui. Deixa eu trocar a key somente para o request_uri para ver se não digitei nada errado. Eu estava me esquecendo de um parâmetro bastante importante. Como nós estamos tratando do fastcgi_cache e não proxy_cache preciso informar também quais códigos de resposta vão ser cacheados por quanto tempo, porque ele não vai ter um padrão assim como proxy_cache.

[03:02] Vamos lá, o que posso informar? Posso vir na documentação dar uma olhada nesse fastcgi_cache_valid, o que preciso informar? Quais códigos http vão ser cacheados, por quanto tempo. Se eu quiser cachear tudo posso mandar um n, e se eu quiser cachear apenas o que ele traz por padrão de sugestão, que são 200, 301 e 302, eu informo somente o tempo.

[03:30] Vamos fazer exatamente isso. Vou transformar em um cache válido somente aquelas respostas padrão, mas não por cinco minutos, vou deixar por um minuto. Vou remover tudo que tem dentro de cache. Meu tmp cache está vazio, e agora quando acesso o meu php info, quando faço um ls de novo tenho o meu 8, que é uma pasta, e se eu fizer um ls recursivo lá dentro tenho o meu arquivo cacheado.

[04:02] Posso tentar acessar ele, por exemplo, para você dar uma olhada no que tem lá dentro. Tem toda aquela minha resposta http já montada com html que foi cacheado nesse arquivo. Com isso repare que agora quando dou enters no meu php fpm e faço a requisição de novo, repare que nenhuma requisição nova aparece aqui.

[04:25] De novo, atualizo, mas nenhuma requisição aparece. Agora, se eu vier e remover o cache de novo, quando fizer a requisição vai aparecer uma nova linha. Com isso o que atinjo? Qual objetivo atinjo? Sempre que alguém acessar meu index, que é um conteúdo que precisa de computação, então ele merece ser cacheado, só que ele não precisa ser cacheado só pelo navegador, pode ser cacheado direto no servidor, eu consigo, ou seja, todos os usuários que acessarem isso não vão precisar esperar o processamento daquelas informações.

[05:02] Eles já podem acessar direto a versão cacheada. Agora conseguimos inclusive enviar informações desse cache, se esse cache está sendo utilizado ou não. Conseguimos mandar essas informações com cabeçalhos http, por exemplo. E isso é especialmente útil quando queremos fazer algum tipo de debug, porque no meu caso é fácil fazer debug, posso ver se a requisição foi feita ou não, estou na mesma máquina, mas às vezes estamos criando um servidor de cache para uma máquina externa onde não temos acesso aos logs, etc.

[05:30] Então podemos enviar esses dados, analisar esses dados de cache. Vamos fazer isso no próximo vídeo.

@@04
Verificando o status

[00:00] Boas-vindas de volta. Vamos dar uma analisada em uma funcionalidade. Não é bem uma funcionalidade, algo que podemos fazer para nos ajudar às vezes a fazer algum tipo de debug, etc. Um detalhe importante de citar é que quando estamos fazendo um servidor de cache precisamos tomar cuidado se temos vários conteúdos sendo modificados na origem, mas alguns ainda estão sendo cacheados e outros não foram cacheados, podemos acabar tendo um conteúdo misturado, isso é perigoso.
[00:30] Na versão comercial do nginx existe uma forma de você remover o cache a partir de uma requisição HTTP, então, por exemplo, sempre que um arquivo foi modificado lá na origem ele pode fazer uma requisição para esse servidor e fazer a limpeza do cache.

[00:48] Na versão gratuita precisamos contar com a inspiração. Mas para o nosso cenário não vai ser problema. E para os cenários onde isso é um problema existe a solução comercial que é simples de implementar. Vamos nessa.

[01:00] O que quero fazer é visualizar através dos cabeçalhos http o status do cache, então vamos lá. Como posso fazer isso? Posso adicionar cabeçalhos http, como já sabemos, vamos na nossa configuração, posso adicionar um header. Quero adicionar o cabeçalho X-cache-Status, e vou adicionar como valor o upstream_cache_status.

[01:30] Esse cabeçalho é um cabeçalho que estou inventando, e já comentei no treinamento anterior que todos os cabeçalhos que são personalizados, que tem significado personalizado dependendo de quem está implementando devem começar com x, isso significa que eles não fazem parte da especificação http. Ou seja, eu que criei esse cabeçalho aqui.

[01:50] E o valor dele é um valor que o nginx consegue trazer para nós, uma variável que o nginx consegue trazer para nós, que é o status do cache para o upstream que está configurado aqui. No nosso cenário, como é um proxy fast CGI não tenho nenhum upstream com load balancer e etc., mas o nome da variável é esse, upstream_cache_status.

[02:15] Com isso consigo adicionar esse cabeçalho, vamos salvar, fazer o reload, torcer para eu não ter escrito nada errado e vamos dar uma olhada no nosso cabeçalho. Quando atualizo, vejo o nosso cache status como expired, o que faz sentido, porque configurei o cache para esperar em um minuto, e já tem mais de um minuto desde o último vídeo até esse.

[02:40] Quando eu fizer uma atualização de novo, como esse cache foi atualizado, quando atualizo tenho um hit, ou seja, agora ele pegou o cache, ele não está utilizando o conteúdo do meu PHP fpm, ele está utilizando o conteúdo que foi cacheado.

[02:58] Agora, se eu remover o meu cache o que vai acontecer? Quando atualizo ele vai dar um miss, ou seja, o conceito de cache miss é que tentei encontrar um cache, mas não achei. Então nesse cenário não tinha cache, e o que acontece? Ele vai no servidor, faz a requisição utilizando o fast CGI, recebe a resposta, faz o cache e me devolve.

[03:25] Quando atualizo de novo tenho o cache hit. Dessa forma além de ver diretamente no PHP fpm que está funcionando consigo mandar a informação para o cliente para que façamos algum tipo de debug, etc.

[03:36] Temos tratado bastante de performance, falamos de performance http, falamos de cache do lado do servidor, inclusive seu propósito, mas também tem um assunto que é muito importante além de performance, que é segurança, e temos utilizado http aqui, como configuramos https no nginx? Porque por enquanto isso ainda não está funcionando. Vamos conversar sobre esse assunto no próximo capítulo.

@@05
Header de status

Neste vídeo nós configuramos o Nginx para sempre enviar na resposta o status do cache para a requisição em questão.
Para que enviamos essa informação?

Para podermos depurar quando necessário.
 
Alternativa correta! Se precisarmos saber se um cache está sendo encontrado ou não, ter um cabeçalho na resposta é uma forma bem fácil de obter essa informação.
Alternativa correta
Para o navegador saber como cachear também.
 
Alternativa correta
Não há motivo real para termos feito isso.
 
Parabéns, você acertou!

@@06
Faça como eu fiz

Chegou a hora de você seguir todos os passos realizados por mim durante esta aula. Caso já tenha feito, excelente. Se ainda não, é importante que você execute o que foi visto nos vídeos para poder continuar com a próxima aula.

Continue com os seus estudos, e se houver dúvidas, não hesite em recorrer ao nosso fórum!

@@07
O que aprendemos?

Nesta aula, aprendemos:
Entendemos o propósito de um servidor de cache
Aprendemos a definir um diretório para armazenar nosso cache
Vimos como usar cache em determinado servidor
Aprendemos a verificar o status do cache em cada requisição

@05-HTTPS

@@01
Funcionamento

[00:00] Boas-vindas de volta a mais um capítulo deste treinamento de nginx, e nós já falamos bastante sobre performance, load balancer, fast CGI, está na hora de falarmos pelo menos um pouco sobre segurança. Quando estamos falando de web o mínimo que temos que fazer em que questões de segurança é ter nosso ambiente conectado e pronto para responder requisições em https.
[00:25] Vamos entender super por alto como funciona esse tal de https, porque lá no treinamento de http já tem um capítulo sobre isso explicando de forma mais didática com mais detalhes. Só vou pincelar aqui caso você não tenha feito aquele treinamento ainda não ficar completamente perdido.

[00:42] Como funciona o http? Vou abrir meu inspecionar elementos. Quando faço alguma requisição temos uma requisição sendo feita e temos uma resposta. Tanto a requisição quanto a resposta trafegam pela rede assim, em texto puro. Imagine que entre o cliente e um servidor já vimos que podem ter várias etapas, inclusive etapas que nem citamos aqui, que têm relação mais com rede do que qualquer outra configuração de web.

[01:11] Por exemplo, um modem, um proxy de rede que já falamos, um proxy reverso, um cdn, só que além dessas coisas que são comuns podemos ter um atacante no meio disso tudo, uma pessoa prestando atenção em tudo que está acontecendo na rede.

[01:26] Eu vou lá, feliz acessar a Alura, quando digito minha senha, coloco meu e-mail, digito minha senha, quando dou enter esse atacante vai conseguir acessar minha senha, vai conseguir ver esses dados que enviei. Isso é péssimo, seria uma péssima prática. Mas quando vemos esse cadeado e um https na frente, sabemos que um processo está acontecendo.

[01:57] Vou super simplificar esse processo baseado numa imagem que peguei da internet para entendermos o que acontece. Só para recapitular como funciona com HTTP sem o https.

[02:10] Todos esses dados que enviei aqui seriam mandados para o servidor direto, em texto puro, então qualquer pessoa no meio, qualquer atacante no meio teria acesso a esses dados. Já quando uso https, o que acontece? Faço a requisição para me conectar a esse servidor e esse servidor devolve um certificado.

[02:35] Quem emite esse certificado é um assunto um pouco complexo, mas existem na internet entidades certificadoras. Existem várias, que eu conheço uma que emite certificados de forma gratuita, e existem várias entidades certificadores que você precisa pagar pelo certificado, tem que renovar de tempo em tempos, etc.

[02:52] Esse certificado garante que esse site é esse site, que ele é confiável, que ele não vai fazer nada de errado com seus dados. A partir disso, com esse certificado que realmente verifica quem é esse servidor, esse site, esse sistema, o navegador recebe esse certificado, a partir desse dado do certificado ele gera uma chave. Essa chave que ele gerou a partir desse certificado ele usa para encriptar a mensagem, para fazer a criptografia da requisição HTTP.

[03:25] E ele vai mandar essa requisição http criptografada. Ou seja, quem está no meio do caminho tentando ouvir, pegar os dados, não vai conseguir recuperar essa informação, porque ela está criptografada. Quando chega no servidor o servidor usa esse mesmo certificado, esses dados de chave que ele tem, que as entidades certificadoras geraram, para descriptografar essa requisição e fazer o processamento normalmente.

[03:54] Essa é a ideia por trás de um https. Mas quando falamos de ambiente de desenvolvimento ou local, ou seja, quando falamos de localhost, não conseguimos falar para a entidade certificadora garantir que localhost é um site confiável, um sistema confiável.

[04:14] Nesses cenários não conseguimos para um ambiente local utilizar dessa forma padrão um https. Mas é muito comum precisarmos ter no nosso ambiente de desenvolvimento https também criptografado para acessar APIs externas, porque algumas APIs verificam que todas as requisições têm que vir de https, para garantir que algumas outras integrações funcionam e também para garantir que nosso servidor web está configurado corretamente dados os certificados.

[04:44] O que vamos fazer nesse capítulo? Ao invés de contatar uma entidade certificadora, etc., nós vamos gerar um certificado. Esse certificado não é válido, o navegador não vai reconhecer ele como válido, mas ele vai funcionar o suficiente para conseguirmos acessar um https. Mas repare que nesse site, o https ele está com um cadeado, a conexão é segura, ele reconhece a entidade certificadora.

[05:14] No nosso caso vamos ter o https, mas não vamos ter o cadeado, porque de novo, não estamos contratando uma entidade certificadora, ou usando alguma gratuita para garantir que esse site realmente é esse site, que não é nada malicioso, etc., mas para ambiente local é o suficiente. Em um ambiente de produção a equipe de segurança vai se reunir e vai decidir qual entidade certificadora utilizar baseado nos critérios, seguranças que cada uma delas fornece, etc., e a partir disso você vai ter os certificados e configurar o nginx.

[05:46] O próximo vídeo é sobre gerar esses certificados, gerar os arquivos necessários para só então configurar o nginx.

@@02
HTTP sem S

Vimos neste vídeo como funciona o HTTPS na prática e qual o seu propósito.
O que acontece com nossos dados quando usamos HTTP , ou seja sem a letra S ao final?

Os dados são criptografados, para impedir a visualização por intermediários.
 
Alternativa correta
Usamos automaticamente um certificado digital para provar a identidade de um site.
 
Alternativa correta
Os dados são transportados em texto puro para o servidor, visível para qualquer um.
 
Alternativa correta! Exato, nossos dados são enviados em texto puro, ficando visível para qualquer um que consiga interceptar nossa conexão!

@@03
Gerando certificado

[00:00] Boas-vindas de volta. Vamos gerar nosso próprio certificado, o que é conhecido como certificado auto assinado. Eu estou gerando, eu estou assinando, eu vou marcar como confiável, etc.
[00:14] Vamos lá, eu vou abrir e preciso do openssl instalado. No meu Mac ele já está instalado por padrão, não precisei fazer nada, mas dependendo do seu sistema operacional você deve precisar instalar alguma coisa ou não. Com o openssl instalado vamos rodar o comando openssl raq, que faz uma requisição de certificado ou quando passamos o parâmetro -k509 ele cria um certificado auto assinado.

[00:40] Estamos dizendo que não precisa criptografar nossa chave, isso vai ficar aqui. Conseguimos colocar uma validade para esse certificado, vou colocar 30 dias para garantir que não aconteça nada de mal, e aqui estamos criando uma nova chave utilizando a criptografia rsa, é uma chave, de 2.048 bits.

[01:02] Nós vamos passar o parâmetro que vai gerar a chave de saída, vou mandar para minha pasta temporária e o nosso certificado de saída. Então temos uma chave e um certificado. Chegando nesse ponto vou executar isso e ele vai me pedir algumas informações para gerar os dados desse certificado.

[01:22] São informações que você enviaria para a entidade certificadora, etc. Vou passar o código do país, um Estado, cidade, você pode colocar dados corretos, eu não vou colocar nada muito válido aqui. Vou colocar o host name como localhost, o e-mail de endereço vai ser example@localhost, e teoricamente os arquivos foram gerados.

[01:51] Quando acesso meu tmp tenho localhost, o certificado e a chave. Temos os dados necessários para continuar. Se eu utilizar esses arquivos na configuração do nginx, isso não vai funcionar ainda, o navegador vai se recusar a usar isso. O que preciso fazer é adicionar na base de dados de certificados no meu computador esses dois certificados falando que sei que não é seguro, mas confia, eu sei o que estou fazendo, pode utilizar.

[02:22] Mal ou bem, mesmo sem exibir o cadeado, deixa eu utilizar isso. Em cada sistema operacional isso vai ser feito de forma diferente. No Mac vou utilizar o comando security, mas tanto no Windows quanto no Linux vocês vão utilizar o certutil, ou seja, utilidade de certificação.

[02:42] Vocês vão utilizar esse comando trocando o foo.crt pelo arquivo crt que geramos. Vou copiar isso, colar, só mudando o caminho para localhost.crt, ele vai dar um aviso, sem problema, e vou copiar esse outro comando, também para tmp/localgost.crt, ele vai pedir minha senha, sem problema, vou digitar, porque estou permitindo que esse certificado seja utilizado como algo confiável.

[03:18] Teoricamente agora tenho minha chave e meu certificado criados, ambos na pasta tmp. Deixei aqui para quando desligar o computador isso ser excluído, mas teoricamente você vai armazenar isso em um local que o nginx tem acesso sempre, etc.

[03:33] Configurado o certificado e a chave precisamos configurar o nginx. Quero fazer com que essa página de parabéns seja disponibilizada através de https. Ou qualquer outro site, pode ser o nosso padrão, 8080, quero que esse seja disponibilizado através de https. Vamos fazer essa configuração no próximo vídeo.

@@04
Para saber mais: Aceitando o certificado

Para garantir que seu navegador reconheça o certificado "auto-assinado" que acabamos de gerar, você pode seguir os passos desta página: Managing security certificates from the console - on Windows, Mac OS X and Linux.

@@05
Configurando Nginx

[00:00] Boas-vindas de volta. Um detalhe importante é que após rodar esse comando security ou o certutil, você precisa fechar todas as janelas do seu navegador, inclusive janelas anônimas, para que quando ele abrir de novo ele verifique essa base de dados de certificados atualizada.
[00:18] Feito isso, o que vamos fazer? Ao acessar https://localhost quero que ele acesse algum dos nossos serviços. Para começar, por que não coloquei uma porta? Porque quero te trazer uma informação relevante. Assim como a porta padrão do http é a porta 80, a porta padrão do https é a 443. Só para termos essa informação vamos configurar essa porta 443.

[00:52] Vou continuar esse de performance, que foi o último que mexi. Vou ter um server e nesse server vou copiar isso tudo, teoricamente não precisaria copiar, podemos conversar sobre isso depois, mas vou copiar aqui, e vamos começar.

[01:10] Primeiro, não vou ouvir essa porta 8005, vou ouvir a porta 443. E vou informar que sempre que essa porta for ouvida preciso utilizar ssl, que é o algoritmo de criptografia, a forma como ele criptografa as conexões https. Mantenho todas essas informações iguais e vou adicionar ssl_certificate, e aquele meu certificado, localhost.crt, e também o ssl_certificate_key, a chave que autentica ele, que é o localhost.key.

[01:47] Teoricamente isso é tudo que preciso. Vamos ver se não digitei nada errado. Vamos tentar acessar o localhost. Quando acesso ele mostra não seguro, mas permite que eu acesse.

[02:00] Da primeira vez que você acessar, deixa eu ver se consigo reproduzir esse comportamento na aba anônima. Não porque ele já salvou. Da primeira vez que você acessar ele vai exibir uma mensagem de que esse site não é seguro, mas ele vai aparecer um botão avançado, e lá você tem a opção de acessar mesmo assim.

[02:17] Deixar eu ver se em outro navegador consigo reproduzir isso para você ver a mensagem. Ele vai exibir que esse certificado não tem um nome válido, sua conexão não é particular, eu não reconheço esse certificado, mas você pode vir em avançado e ir para localhost mesmo assim, e acessamos o endereço.

[02:36] Dessa forma conseguimos conectar, conseguimos utilizar https. Um detalhe, eu poderia facilmente configurar para que sempre que eu acessar através dessa porta, que seria a padrão, a 80, faço um redirecionamento, mando um cabeçalho de redirecionamento para esse servidor. Isso é comum para que nós forcemos a utilização do https, é algo bastante comum, mas nesse caso vou manter assim para termos os dois serviços separados.

[03:06] Dessa forma conseguimos gerar um certificado https, assinar ele, marcar como confiável no nosso computador e configurar o nginx através do certificado e a chave desse certificado, ambos com essas configurações simples. Claro, lembrando de ouvir a porta correta, se for o caso de não querermos que digite a porta, e sempre adicionar o ssl.

@@06
Faça como eu fiz

Chegou a hora de você seguir todos os passos realizados por mim durante esta aula. Caso já tenha feito, excelente. Se ainda não, é importante que você execute o que foi visto nos vídeos para poder continuar com os próximos cursos que tenham este como pré-requisito

Continue com os seus estudos, e se houver dúvidas, não hesite em recorrer ao nosso fórum!

@@07
O que aprendemos?

Nesta aula, aprendemos:
Conhecemos o funcionamento do HTTPS
Aprendemos a gerar um certificado SSL
Configuramos o Nginx para usar nosso certificado e chave para servir por HTTPS

@@08
Conclusão

[00:00] Parabéns por chegar ao final deste treinamento de nginx, onde nos aprofundamos um pouco nos nossos conhecimentos. Começamos mexendo um pouco no nosso load balancer, vamos dar uma olhada no arquivo dele, porque vimos além de outros algoritmos de load balancer servidores de backup, vimos sobre o tempo de falha, aprendemos sobre round Robin, waited round Robin, vimos sobre o least connections, vimos bastante coisa.
[00:28] Além disso, aprendemos sobre o fast CGI, e no final das contas configuramos um fast CGI proxy, como se fosse um proxy reverso para um fast CGI, para um servidor que recebe requisições desse tipo, utilizando esse protocolo, que é um protocolo de comunicação entre processos, basicamente, através da rede, e já pegando um gancho aqui também aprendemos a utilizar cache do lado do servidor, para transformar no nginx em um servidor de cache, e não só um load balancer, não só um proxy reverso, não só um servidor web.

[01:02] Estamos vendo quanta coisa o nginx pode fazer. Mas antes disso aprendemos algumas coisas bem interessantes sobre performance. Nesse ponto de performance vimos sobre compressão dos nossos dados através do gzip e vimos como informar o que será comprimido. Vimos sobre keep alive connections, manter as conexões abertas, falamos sobre cache http com a diretiva expires, falamos de cache control public, etc.

[01:30] No final do treinamento batemos um papo bem rápido sobre https, ssl, geração de certificados e essa parte de segurança, que é o mínimo que precisamos saber quando vamos colocar algo no ar. Claro que não utilizamos uma entidade certificadora aqui para não ter custos ou super complicar, e também para não precisarmos de um domínio real, porque estamos usando o localhost, ou seja, nossa própria máquina.

[01:54] Depois de tudo isso espero que você tenha aproveitado bastante, e caso tenha ficado com alguma dúvida não hesite, você pode abrir um tópico no fórum, eu tento responder pessoalmente sempre que possível, mas quando não consigo nós temos uma comunidade bem solicita de alunos, moderadores, instrutores, e com certeza alguém vai conseguir te ajudar. Mais uma vez parabéns por ter chegado até o final desse treinamento, obrigado por ter me aguentado até aqui, e espero te ver em outros conteúdos da Alura. Forte abraço.