# Git Cheat Sheet

Lista de comandos uteis, que uso bastante (alguns nem tanto), no meu dia-a-dia.

## Configuração do git

Para toda nova instalação, é necessário informar ao git algumas coisas que serão utilizadas nos seus commits. Seguem abaixo:
```sh
# configuração do meu nome e e-mail
git config --global user.name "John Doe"
git config --global user.email "john.doe@gmail.com"
```

## Inicialização de projeto
```sh
# Inicialização local de um novo repositório. Tranforma qualquer pasta em um repositório git.
git init .

# Clona um diretório remoto para a pasta atual. Você pode alterar o nome da pasta onde será
# criada sua cópia, adicionando o nome dela logo à frente do comando.
git clone git@git.raroacademy.com.br:john-doe/Aulas.git
```

## Rotina de trabalho com o git

### Finalização das tarefas
Quando finalizo minha tarefa, ou estou em um ponto onde quero salvar o progresso, faço os seguintes passos:

```sh
# apresenta o status dos arquivos na pasta correte (.). É possível verificar o status em outras pastas,
# substituindo o "." pela navegação de pastas, ex.: git status ../src/aula.ts
git status .
# comando para ver as modificações do arquivo aula.ts, em comparação ao último stage (git add) deste arquivo.
# Para ver as modificações existentes no arquivo já staged com a versão commitada, use o `git diff .--staged`
git diff aula.ts
# adiciona o arquivo aula.ts para o stage. Estar no stage significa que não está salvo, mas selecionado para ser commitado em seguida.
# Repare que estou usando um fluxo de diff/add para os arquivos. Sugiro que sempre, ao commitar, vc mantenha um padrão parecido,
# para garantir que somente o que você pretende mesmo salvar seja adicionado.
git add aulta.ts
# cria um novo commit (registro de modificações) no git. Como o git trabalha em um modelo incremental de modificações,
# este comando adiciona mais um registro ao histórico.
git commit -m "descreve o que foi feito neste commit"
# verifica se há modificações no repositório remoto (origin) para a branch (main). Caso haja, este comando baixa as modificações
# e tenta conciliar com a versão local.
git pull origin main
# envia as modificações do repositório local para o repositório (origin), para a branch main.
git push origin main
```

> 💡 **Dica**: a mensagem do commit é fundamental para que você e o restante da equipe saibam o qeu foi feito exatamente naquele commit. Cada projeto ou empresa tem seu próprio padrão de definição das mensagens. Mas como regra geral, eu tento descrever, de forma breve mas clara, tudo o que fiz naqeule commit. Por padronização, sempre escrevo a mensagem em terceira pessoal do presente. Ex.: "adiciona o arquivo de configuração do projeto", "corrige o bug de login".

## Branches

### comandos básicos

```sh
# lista todas as branches locais. Para ver também as branches remotas, use o modificador `-a`
git branch
# cria uma nova branch, a partir da branch que estou no momento.
git checkout -b nova-branch
# sai da branch atual, e vai para a branch indicada. Sugiro que você resolva toda a working area (commite todos os arquivos)
# antes de trocar de branch.
git checkout develop
# o checkout também pode voltar diretamente para commits. Este número hexadecimal à frente do comando são os 6 primeiros
# digitos do commit, que encontrei com o comando `git log`
git checkout 5f57a0
# deleta uma branch local. Use com cuidado, pois não há como recuperar uma branch deletada, a menos que ela exista em repositório
# remoto.
git branch -D feature/nova-tarefa
# faz o merge da branch atual com a branch indicada. O merge é o comando que faz a integração de duas branches.
git merge develop
```

### rotina de inicialização de uma nova tarefa:

considerando que devo iniciar minhas tarefas a partir da branch develop,

```sh
git checkout develop
git pull origin develop
git checkout -b feature/nova-tarefa
```

### rotina de troca de branch

Em alguns momentos, é necessário que você interrompa o que está fazendo, resolva problemas em outra branch, e volte para seu trabalho original. Caso o trabalho atual esteja em uma situação que você possa fazer o commit, sugiro que seja feito o fluxo add/commit/push. Caso existam coisas que você não deseja ainda entregar, pois está em desenvolvimento e precisa de alguma limpeza, você pode utilizar a rotina:

```sh
# adiciono todas as alterações, sem precisar ainda verificar o conteúdo delas. Em seguida, faço um commit com uma mensagem
# que me ajude a identificar que o trabalho não estava finalizado, e por isso devo retomar. A mensagem "wip" (work in progress),
# neste caso, é minha escolha, pois é algo que escrevo de forma bem ágil, e que sei facilmente que posso desfazer
git add .
git commit -m "wip"
# vou à branch que preciso atuar, faço todo o trabalho necessário nela.
git checkout feature/tarefa-que-preciso-resolver\
# Em seguida, retomo à minha branch original
git checkout feature/tarefa-anterior

# faço um git log, para verificar o histórico, e ver se o commit "wip" é o ultimo estado desta branch. Se for o caso,
# removo este commit do histórico, retomo ele para a minha working area, e volto ao trabalho normalmente.
git log
git reset HEAD~1
```

### rotina de merge seguro

Algumas vezes, é necessário que você faça o merge entre duas branches manualmente. Acontece, por exemplo, quando você precisa criar um pull request entregando sua tarefa, porém há conflitos entre as versões.

Dado que a ação de merge pode causar conflitos, que podem te dar bastante trabalho para resolver, sugiro que você faça este merge em uma branch temporária, e depois concilie o trabalho na sua versão. Supondo, por exemplo, que eu queria resolver conflitos existentes entre a branch `develop` e minha tarefa atual, na branch `feat/login-usuario-via-google`

```sh
# vou à branch develop e busco todo o conteúdo dela. Retomo a branch atual.
git checkout develop
git pull origin develop
git checkout feat/login-usuario-via-google
# a partir da branch atual, crio uma nova branch, a ser utilizada para a resolução do conflito.
# para encontrar com mais facilidade esta branch, sempre uso o padrão de nome `merge/<branch-origem>_<branch-destino>`
git checkout -b merge/develop_feat/login-usuario-via-google
# faço o merge de develop para esta branch. Como ela saiu de `feat/login-usuario-via-google`, este comando
# vai integrar as duas branchs em uma terceira, garantindo que as duas originais não sejam comprometidas.
git merge develop
# caso haja conflitos, resolva-os, e faça o commit das resoluções

# remoto a branch da minha tarefa e faço o merge da branch temporária de resolução de conflitos. Como todos eles foram
# resolvidos, não haverá conflitos.
git checkout feat/login-usuario-via-google
git merge merge/develop_feat/login-usuario-via-google
# deleto a branch temporária.
git branch -D merge/develop_feat/login-usuario-via-google
```

### Remoção de conteúdo

> 💣 os comandos abaixo são perigosos, pois você pode perder grande parte de trabalho já executado, se feito de forma incorreta. Utilize com atenção redobrada.

```sh
# caso eu queira descartar as alterações atuais no arquivo aula.ts, e voltar para a versão **staged**, uso o comando abaixo.
# Importante ter cuidado com um comando `git checkout -- .`, por exemplo. Este comando deverá descartar todas as modificações
# feitas de forma irreversível.
git checkout -- aula.ts

# caso existam modificação em stage (já fiz um git add), e ainda pretendo descartar:
git reset -- aula.ts
git checkout -- aula.ts

# reverte um commit, voltando o conteúdo deste para o "working area". Para reverter dois commits, utilize o `HEAD~2`,
# e assim por diante.
# Para reverter e elimitar todo o conteúdo definitivamente, utilize o comando `git reset --hard HEAD~1`
git reset HEAD~1
```

## Comandos úteis

```sh
# mostra o histórico de commits de uma branch. Comando muito util para entender o que foi feito, e quando.
# em algumas situações, quando é preciso voltar a versões anteriores, este comando nos ajuda a encontrar os
# pontos onde podemos voltar.
git log
```

```sh
# apresenta os principais comandos do git, breve explicação. Para ver todos os comandos, use o modificador --all
git help
# lista todas as opções do comando git add. você pode usar também a opção `-h`
git add --help
```

## fontes
- [Gitlab Git Cheat Sheet](https://about.gitlab.com/images/press/git-cheat-sheet.pdf)
- [Atlassian Git Cheat Sheet](https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet)
- [Github Education Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)