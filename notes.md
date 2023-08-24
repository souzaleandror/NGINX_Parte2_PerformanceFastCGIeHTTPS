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