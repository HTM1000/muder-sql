# SQL Murder Mystery
 - Aqui farei um passo a passo como resolver o jogo SQL do site -> https://mystery.knightlab.com/
   Nele somos um detetive que irá desvendar um assassinato e para isso precisamos ir consultando dados no banco para encontrar cada vez novas pistas e conseguir inserir a solução correta.
   O site está todo em inglês, caso você for resolver precisa escrever as palavras nesta linguagem, mas neste documento irei colocar em portugues as dicas mas o comando SQL em inglês. Vou deixar as tabelas em um arquivo png.

## CASO VOCE QUEIRA JOGAR, NÃO OLHE O PASSO A PASSO A SEGUIR

# Primeiro passo
 - Lendo a primeira dica que ele nós da ->
   Um crime ocorreu e o detetive precisa de sua ajuda. O detetive lhe deu o relatório da cena do crime, mas de alguma forma você o perdeu. Você lembra vagamente que o crime foi um assassinato (muder) ocorrido em 15 de janeiro de 2018 e que ocorreu em SQL City. Comece recuperando o relatório da cena do crime correspondente do banco de dados do departamento de polícia.
   Com essa dica conseguimos saber que ele foi um crime do tipo "murder" ocorrido em 15 janeiro de 2018 e local foi em SQL City gerando assim nossa primeira busca

   ## SELECT * FROM crime_scene_report WHERE type = 'murder' AND Date = 20180115 AND city = 'SQL City'
   com ela conseguimos nossa primeira pista que é a seguinte -> 
   Imagens de segurança mostram que havia duas testemunhas. A primeira testemunha mora na última casa da "Northwestern Dr". A segunda testemunha, chamada Annabel, mora em algum lugar na "Franklin Ave".

# Segundo passo
 - Agora precisamos descobrir quem mora na última casa da "Northwestern Dr" e aonde a segunda          testemunha mora. Rodando esta query -> 
   ## SELECT * FROM person WHERE name LIKE 'Annabel%' AND address_street_name = 'Franklin Ave'
   Com essa primeira query conseguimos todos os dados restantes da Annabel Miller
   ID - 16371 
   License_Id - 490173
   address_number - 103
   ssn - 318771143
   e para conseguir os dados da outra testemunha, utilizei a seguinte query -> 
   ## SELECT MAX(address_number) FROM person WHERE address_street_name = 'Northwestern Dr'
   Fiz primeiramente esta query por supor que a última casa desta rua seja a com maior número assim descobrindo que a casa seria número -> 4919, agora ficando fácil de descobrir os dados da última pessoa com a seguinte query -> 
   ## SELECT * FROM person WHERE address_street_name = 'Northwestern Dr' AND address_number = 4919
   Conseguindo assim todos os dados faltantes -> 
   id -> 14887
   name -> Morty Schapiro	
   license_id -> 118009
   ssn -> 111564949

# Terceiro passo
 - Precisamos neste próximo passo pegar o depoimentos dessas duas testemunhas para o crime que ocorreu naquele dia, para conseguir isso, utilizei esta query -> 
   ## SELECT * FROM interview WHERE person_id = 16371 OR person_id = 14887
   Assim ganhamos acesso as duas entrevistas

   ## Annabel Miller 
   Eu vi o assassinato acontecer e reconheci o assassino da minha academia quando estava malhando na semana passada, no dia 9 de janeiro.
   ## Morty Schapiro
   Ouvi um tiro e vi um homem sair correndo. Ele tinha uma bolsa "Get Fit Now Gym". O número de membro na bolsa começava com “48Z”. Somente membros ouro possuem essas sacolas. O homem entrou em um carro com placa que incluía “H42W”.

   Analisando os dois depoimentos podemos notar que o rapaz provavelmente faça academia com certeza e que sua bolsa possui um código "48Z". Homem entrou em um carro com a placa "H42W"
  
# Quarto passo
 - Acredito que o mais fácil seja consultar os checkins do dia 09 de janeiro na academia e verificar quem poderia ser nosso suspeito, utilizando esta query
   ## SELECT * FROM get_fit_now_check_in WHERE check_in_date = 20180109
   Fazendo está consulta me retornaram 10 registros, e analisando eles dois me chamaram atenção por possuirem ids da academia que correspondem ao número da bolsa que o Morty identificou que são -> 
   Id 1 -> 48Z55
   Id 2 -> 48Z7A
   Irei salvar aqui tambem o tempo que eles ficaram na academia já que eu consegui estas informações -> 
   check_in_time 1 -> 1530
   check_out_time 1 -> 1700
   check_in_time 2 -> 1600
   check_out_time 2 -> 1730

# Quinto passo
 - Agora por termos a informação que um deles pode ser gold, irei fazer um query para ver qual é o status deles na academia
  ## SELECT * FROM get_fit_now_member WHERE id = '48Z55' OR id = '48Z7A'
  Bom, esse passo foi meio ruim pois os dois membros são Gold, mas pelo menos descobrimos os outros dados deles, para conseguir acessar as outras tabelas -> 
  ## Jeremy Bowers	
  person_id - 67318
  membership_start_date - 20160101
  membership_status - gold
  ## Joe Germuska	
  person_id - 28819
  membership_start_date - 20160305
  membership_status - gold
  Aparentemente os dois possuem nomes de homens também então nao podemos descartas nenhum dos dois,

# Sexto Passo
 - Bom, sobrou somente a dica que ele entrou em um carro com a placa "H42W", vou fazer duas querys para buscar os dados do carro deles com essas queries -> 
   ## SELECT * FROM person WHERE id = 67318 OR id = 28819
   Nessa primeira consegui as duas licencas de direção que eu queria e outros dados
   ## Jeremy Bowers	
   license_id -> 423327
   address_number -> 530
   address_street_name -> Washington Pl, Apt 3A	
   ssn -> 871539279
   ## Joe Germuska	
   license_id -> 173289
   address_number -> 111
   address_street_name -> Fisk Rd
   ssn -> 138909730
   ## SELECT * FROM drivers_license WHERE id = 173289 OR id = 423327
   Realizando esta query notasse que só retornou os dados do ID 423327 correspondente ao Jeremy e analisando os dados me retornaram o seguinte -> 
   plate_number -> 0H42W2
   O ultimo dado que faltava e então descobrimos nosso suspeito!

# Sétimo passo
  - Inserindo os dados para validar se nossa resposta está correta ->
    ## INSERT INTO solution VALUES (1, 'Jeremy Bowers'); 
    ## SELECT value FROM solution;
    O jogo nos retorna a mensagem de sucesso! Mas ainda não acabou...
    Parabéns, você encontrou o assassino! Mas espere, tem mais... Se você acha que está pronto para um desafio, tente consultar a transcrição da entrevista do assassino para encontrar o verdadeiro vilão por trás desse crime. Se você se sentir especialmente confiante em suas habilidades em SQL, tente concluir esta etapa final com no máximo duas consultas. Use esta mesma instrução INSERT com seu novo suspeito para verificar sua resposta.

# Oitavo passo
  - Assim como foi solicitado iremos ver o depoimento do assassino para acharmos o verdadeiro culpado por armar este crime, como ja fizemos antes, iremos ver a tabela de depoimentos -> 
   ## SELECT * FROM interview  WHERE person_id = 67318
   Obtivemos o seguinte depoimentos -> 
   Fui contratado por uma mulher com muito dinheiro. Não sei o nome dela, mas sei que ela tem cerca de 5'5" (65") ou 5'7" (67"). Ela é ruiva e dirige um Tesla Model S. Sei que ela participou do SQL Symphony Concert três vezes em dezembro de 2017.
   Agora precisamos conseguir achar essa mulher ruiva que dirige o Tesla

# Nono passo
  - Agora irei pesquisar no evento de acordo com a data, nome e quem foi mais de 3 vezes neste mesmo intervalo de data com a seguinte query -> 
   ## SELECT * FROM facebook_event_checkin 
   ## WHERE event_name = 'SQL Symphony Concert' AND date >= 20171200 AND date < 20180100 
   ## GROUP BY person_id 
   ## HAVING COUNT(*) = 3 
   Assim consigo o id dos suspeitos -> 
   person_id 1 -> 24556
   person_id 2 -> 99716

# Décimo passo
  - Poderiamos descobrir o id da carteira de motorista dela e após isso procurar os dados nesta tabela mas isso geraria 3 querys após o depoimentos do assassino, então para completarmos com duas querys, eu precisarei dar um join com a tabela pessoa e carteira de motorista  -> 
   ## SELECT * FROM person
   ## JOIN drivers_license ON person.license_id == drivers_license.id 
   ## WHERE person.id = 24556 OR person.id = 99716
   assim descobrindo nossa verdadeira mandante do assassinato
   # Miranda Priestly	- 99716

# Décimo primeiro passo
  - Inserindo novamente mais uma pessoa na lista e consultando se está correta
     ## INSERT INTO solution VALUES (1, 'Miranda Priestly'); 
     ## SELECT value FROM solution;
    O jogo retorna a mensagem de sucesso!
    Parabéns, você encontrou o cérebro por trás do assassinato! Todos em SQL City o consideram o maior detetive de SQL de todos os tempos. Hora de abrir o champanhe!

## Finalizando
  - Assim finalizamos um breve joguinho tutorial de SQL. Simples, rápido e divertido, nota 10!
   