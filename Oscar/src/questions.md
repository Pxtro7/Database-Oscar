# Oscar - Exercícios MongoDB

Bem-vindo à base de dados do **Oscar**!

O Oscar é a premiação mais prestigiada do cinema mundial, realizada anualmente desde 1929 pela Academia de Artes e Ciências Cinematográficas. Nesta base de dados, você encontrará registros históricos de indicados e vencedores de diversas categorias ao longo de quase 100 anos de história do cinema.

Nestes exercícios, você vai explorar o banco de dados MongoDB do Oscar e responder perguntas que revelam insights fascinantes sobre a história do cinema, tendências de premiação, e momentos marcantes da indústria cinematográfica.

---

## 📊 Nível 1: Primeiros Passos

### Conhecendo a Base de Dados

**1.1** Quantos registros existem na coleção de indicados ao Oscar? R: **10889**
`db.filmes.countDocuments()`

**1.2** Quais são as diferentes categorias de premiação que existem no banco de dados? Liste todas as categorias únicas.
`db.filmes.distinct("categoria")
`

**1.3** Qual foi o primeiro ano de cerimônia do Oscar registrado na base? R: **1928**
`db.filmes.aggregate([
{
$group: {
_id: null,
anoMinimo: { $min: "$ano_cerimonia" }
}
}
])`

**1.4** Qual foi o último ano de cerimônia registrado na base? R: **2025**

`db.filmes.aggregate([
{
$group: {
_id: null,
anoMaximo: { $max: "$ano_cerimonia" }
}
}
])`

**1.5** Quantas cerimônias do Oscar estão registradas no total? R: **97 cerimonias**
`db.filmes.aggregate([
{
$group: {
_id: null,
QntdCerimonia: { $max: "$cerimonia" }
}
}
])`

**1.6** Atualize os registros da tabela com os dados do Oscar 2025 (pesquise os vencedores e adicione-os).

---

## 🎬 Nível 2: Explorando Categorias

**2.1** Quantas indicações existem para cada categoria? Agrupe por categoria e ordene da mais frequente para a menos frequente.
`db.filmes.aggregate([
{
$group: {
_id: "$categoria",
total_indicacoes: { $sum: 1 }
}
},
{
$sort: {
total_indicacoes: -1
}
},
{
$project: {
_id: 0,
categoria: "$_id",
total_indicacoes: 1
}
}
])`

**2.2** Qual categoria teve mais indicações ao longo da história do Oscar? R: **DIRECTING**
`db.filmes.aggregate([
{
$group: {
_id: "$categoria",
total_indicacoes: { $sum: 1 }
}
},
{
$sort: {
total_indicacoes: -1
}
},
{
$group: {
_id: null,
resultados_ordenados: {
$push: {
categoria: "$_id",
total_indicacoes: "$total_indicacoes"
}
}
}
},
{
$project: {
_id: 0,
mais_indicacoes: { $first: "$resultados_ordenados" },
menos_indicacoes: { $last: "$resultados_ordenados" }
}
}
])
`

**2.3** Qual categoria teve menos indicações ao longo da história? R: **AWARD OF COMMENDATION**

`db.filmes.aggregate([
{
$group: {
_id: "$categoria",
total_indicacoes: { $sum: 1 }
}
},
{
$sort: {
total_indicacoes: -1
}
},
{
$group: {
_id: null,
resultados_ordenados: {
$push: {
categoria: "$_id",
total_indicacoes: "$total_indicacoes"
}
}
}
},
{
$project: {
_id: 0,
mais_indicacoes: { $first: "$resultados_ordenados" },
menos_indicacoes: { $last: "$resultados_ordenados" }
}
}
])
`

**2.4** A partir de que ano a categoria "ACTRESS" deixou de existir? (Dica: procure a última cerimônia com essa categoria). R: **1976**

`db.filmes.aggregate([
{
$match: {
categoria: "ACTRESS"
}
},
{
$group: {
_id: "$categoria",
ultimo_ano: { $max: "$ano_cerimonia" }
}
},
{
$project: {
_id: 0,
categoria: "$_id",
ultima_cerimonia: "$ultimo_ano"
}
}
])`

**2.5** Quais categorias existiam na primeira cerimônia (1928) e não existem mais hoje?   
****categorias_descontinuadas:
'ART DIRECTION',
'ACTRESS',
'ENGINEERING EFFECTS',
'OUTSTANDING PICTURE',
'UNIQUE AND ARTISTIC PICTURE',
'DIRECTING (Dramatic Picture)',
'WRITING (Adaptation)',
'ACTOR',
'WRITING (Original Story)',
'WRITING (Title Writing)',
'SPECIAL AWARD',
'DIRECTING (Comedy Picture)'****


`db.filmes.aggregate([
{
$facet: {
"categoriasPrimeiraCerimonia": [
{ $match: { cerimonia: 1 } },
{ $group: { _id: "$categoria" } }
],
"categoriasHoje": [
{ $match: { cerimonia: 97 } },
{ $group: { _id: "$categoria" } }
]
}
},
{
$project: {
categorias_descontinuadas: {
$setDifference: [
"$categoriasPrimeiraCerimonia._id",
"$categoriasHoje._id"
]
}
}
}
])`

**2.6** Liste todas as categorias que contêm a palavra "DIRECTING" no nome. R:  **DIRECTING, DIRECTING (Comedy Picture), DIRECTING (Dramatic Picture)**

`db.filmes.distinct(
"categoria",
{ categoria: { $regex: "DIRECTING" } }
)`

---

## 🌟 Nível 3: Atores e Atrizes Famosos

### Natalie Portman

**3.1** Quantas vezes Natalie Portman foi indicada ao Oscar? R: **3 indicações**
`db.filmes.find({"nome_do_indicado": "Natalie Portman"}).count()
`
**3.2** Quantos Oscars Natalie Portman ganhou? R: **1 vitória**

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $regex: "Natalie Portman", $options: "i" },
vencedor: { $in: [1, "true"] }
}
},
{
$count: "total_de_vitorias"
}
])`

**3.3** Em quais anos e por quais filmes Natalie Portman foi indicada? 

R: 
****ano_da_cerimonia: 2005,
filme: 'Closer',
categoria: 'ACTRESS IN A SUPPORTING ROLE'****

****ano_da_cerimonia: 2011,
filme: 'Black Swan',
categoria: 'ACTRESS IN A LEADING ROLE'****

****ano_da_cerimonia: 2005,
filme: 'Closer',
categoria: 'ACTRESS IN A SUPPORTING ROLE'****

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $regex: "Natalie Portman", $options: "i" }
}
},
{
$sort: {
ano_cerimonia: 1
}
},
{
$project: {
_id: 0,
ano_da_cerimonia: "$ano_cerimonia",
filme: "$nome_do_filme",
categoria: "$categoria"
}
}
])`

**3.4** Liste todas as indicações de Natalie Portman mostrando: ano, categoria, filme e se venceu.

R:
**ano: 2005,
categoria: 'ACTRESS IN A SUPPORTING ROLE',
filme: 'Closer',
venceu: 'false'**

**ano: 2011,
categoria: 'ACTRESS IN A LEADING ROLE',
filme: 'Black Swan',
venceu: 'true'**

**ano: 2017,
categoria: 'ACTRESS IN A LEADING ROLE',
filme: 'Jackie',
venceu: 'false'**

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $regex: "Natalie Portman", $options: "i" }
}
},
{
$sort: {
ano_cerimonia: 1
}
},
{
$project: {
_id: 0,
ano: "$ano_cerimonia",
categoria: "$categoria",
filme: "$nome_do_filme",
venceu: "$vencedor"
}
}
])`

### Viola Davis

**3.5** Quantas vezes Viola Davis foi indicada ao Oscar? R: **4 indicações**
`db.filmes.find({"nome_do_indicado": "Viola Davis"}).count()
`


**3.6** Quantos Oscars Viola Davis ganhou? R: **1 vitória**

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $regex: "Viola Davis", $options: "i" },
vencedor: { $in: [1, "true"] }
}
},
{
$count: "total_de_vitorias"
}
])`

**3.7** Por quais filmes Viola Davis foi indicada?

R:
**ano: 2009,
filme: 'Doubt',
venceu: 'false'**

**ano: 2012,
filme: 'The Help',
venceu: 'false'**

**ano: 2017,
filme: 'Fences',
venceu: 'true'**

**ano: 2021,
filme: "Ma Rainey's Black Bottom",
venceu: 'false'**

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $regex: "Viola Davis", $options: "i" }
}
},
{
$sort: {
ano_cerimonia: 1
}
},
{
$project: {
_id: 0,
ano: "$ano_cerimonia",
filme: "$nome_do_filme",
venceu: "$vencedor"
}
}
])`

### Amy Adams

**3.8** Amy Adams já ganhou algum Oscar? R: **Não teve nenhuma vitória no oscar**
`db.filmes.find({"nome_do_indicado": "Amy Adams"})
`

**3.9** Quantas vezes Amy Adams foi indicada sem ganhar? R: **6 indicações**
`db.filmes.find({"nome_do_indicado": "Amy Adams"})
`

### Denzel Washington

**3.10** Denzel Washington já ganhou algum Oscar? R: **2 vitórias**

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $regex: "Denzel Washington", $options: "i" },
vencedor: { $in: [1, "true"] }
}
},
{
$count: "total_de_vitorias"
}
])`

**3.11** Quantas vezes Denzel Washington foi indicado ao Oscar? R: **9 indicações**

`db.filmes.find({"nome_do_indicado": "Denzel Washington"}).count()
`

**3.12** Liste todos os Oscars que Denzel Washington ganhou (ano, categoria, filme).
R:   **ano: 1990,
categoria: 'ACTOR IN A SUPPORTING ROLE',
filme: 'Glory'**

**ano: 2002,
categoria: 'ACTOR IN A LEADING ROLE',
filme: 'Training Day'**

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $regex: "Denzel Washington", $options: "i" },
vencedor: { $in: [1, "true"] }
}
},
{
$sort: {
ano_cerimonia: 1
}
},
{
$project: {
_id: 0,
ano: "$ano_cerimonia",
categoria: "$categoria",
filme: "$nome_do_filme"
}
}
])`

---

## 🏆 Nível 4: Vencedores Históricos

**4.1** Quem ganhou o primeiro Oscar para Melhor Atriz (ACTRESS)? Em que ano e por qual filme?

R:   **quem_ganhou: 'Janet Gaynor',
ano: 1928,
filme: '7th Heaven'**

`db.filmes.aggregate([
{
$match: {
cerimonia: 1,
categoria: "ACTRESS",
vencedor: { $in: [1, "true"] }
}
},
{
$project: {
_id: 0,
quem_ganhou: "$nome_do_indicado",
ano: "$ano_cerimonia",
filme: "$nome_do_filme"
}
}
])`

**4.2** Quem ganhou o primeiro Oscar para Melhor Ator (ACTOR)? Em que ano e por qual filme?

R:   **quem_ganhou: 'Emil Jannings',
ano: 1928,
filme: 'The Last Command'**

`db.filmes.aggregate([
{
$match: {
cerimonia: 1,
categoria: "ACTOR",
vencedor: { $in: [1, "true"] }
}
},
{
$project: {
_id: 0,
quem_ganhou: "$nome_do_indicado",
ano: "$ano_cerimonia",
filme: "$nome_do_filme"
}
}
])`

**4.3** Quantos vencedores existem ao todo na base de dados?

R: **2484 vencedores**

`db.filmes.find({"vencedor": "true"}).count()
`

**4.4** Liste todos os filmes que ganharam o Oscar de Melhor Filme (categoria "OUTSTANDING PICTURE" ou "BEST PICTURE").

`db.filmes.aggregate([
{
$match: {
categoria: { $in: ["BEST PICTURE", "OUTSTANDING PICTURE"] },
vencedor: { $in: [1, "true"] }
}
},
{
$sort: {
ano_cerimonia: 1
}
},
{
$project: {
_id: 0,
ano: "$ano_cerimonia",
filme: "$nome_do_filme",
categoria_premio: "$categoria"
}
}
])`

**4.5** Quantos filmes diferentes já ganharam o Oscar?

R: **1339 filmes vencedores**

`db.filmes.aggregate([
{
$match: {
vencedor: { $in: [1, "true"] }
}
},
{
$group: {
_id: "$nome_do_filme"
}
},
{
$count: "total_filmes_diferentes_vencedores"
}
])`

---

## 🎭 Nível 5: Análise de Indicações

**5.1** Quais atores/atrizes foram indicados mais de uma vez? Liste o nome e o número de indicações.

`db.filmes.aggregate([
{
$match: {
categoria: {
$in: [
"ACTOR",
"ACTRESS",
]
}
}
},
{
$group: {
_id: "$nome_do_indicado",
numero_de_indicacoes: { $sum: 1 }
}
},
{
$match: {
numero_de_indicacoes: { $gt: 1 }
}
},
{
$sort: {
numero_de_indicacoes: -1
}
},
{
$project: {
_id: 0,
nome: "$_id",
numero_de_indicacoes: 1
}
}
])`

**5.2** Qual ator ou atriz tem o maior número de indicações na história do Oscar?
R:  **numero_de_indicacoes: 11,
nome: 'Katharine Hepburn'**

**numero_de_indicacoes: 11,
nome: 'Bette Davis'**

`db.filmes.aggregate([
{
$match: {
categoria: {
$in: [
"ACTOR",
"ACTRESS",
]
}
}
},
{
$group: {
_id: "$nome_do_indicado",
numero_de_indicacoes: { $sum: 1 }
}
},
{
$sort: {
numero_de_indicacoes: -1
}
},
{
$limit: 3
},
{
$project: {
_id: 0,
nome: "$_id",
numero_de_indicacoes: 1
}
}
])`



**5.3** Quais atores foram indicados mais de 3 vezes mas nunca ganharam?

`db.filmes.aggregate([
{
$match: {
categoria: {
$in: [
"ACTOR",
"ACTRESS",
]
}
}
},
{
$group: {
_id: "$nome_do_indicado",
total_indicacoes: { $sum: 1 },
total_vitorias: {
$sum: {
$cond: [ { $in: ["$vencedor", [1, "true"]] }, 1, 0 ]
}
}
}
},
{
$match: {
total_indicacoes: { $gt: 3 },
total_vitorias: 0             
}
},
{
$sort: {
total_indicacoes: -1
}
},
{
$project: {
_id: 0,
nome: "$_id",
indicacoes: "$total_indicacoes",
vitorias: "$total_vitorias"
}
}
])`

**5.4** Encontre todos os artistas que foram indicados em categorias diferentes (ex: ator e diretor).

`db.filmes.aggregate([
{
$group: {
_id: "$nome_do_indicado",
categorias_unicas: { $addToSet: "$categoria" }
}
},
{
$addFields: {
num_categorias_diferentes: { $size: "$categorias_unicas" }
}
},
{
$match: {
num_categorias_diferentes: { $gt: 1 }
}
},
{
$sort: {
num_categorias_diferentes: -1
}
},
{
$project: {
_id: 0,
nome_do_artista: "$_id",
categorias_indicadas: "$categorias_unicas"
}
}
])`


**5.5** Quantos indicados têm exatamente 1 indicação na história?

R: **Indicados_com_apenas_uma_indicacao: 5641**

`db.filmes.aggregate([
{
$group: {
_id: "$nome_do_indicado",
total_indicacoes: { $sum: 1 }
}
},
{
$match: {
total_indicacoes: 1
}
},  
{
$count: "indicados_com_apenas_uma_indicacao"
}
])`

---

## 🎥 Nível 6: Análise de Filmes

### Toy Story

**6.1** A série de filmes Toy Story ganhou Oscars em quais anos?

`db.filmes.aggregate([
{
$match: {
nome_do_filme: { $regex: /Toy Story/, $options: "i" },
vencedor: { $in: [1, "true"] }
}
},
{
$sort: {
ano_cerimonia: 1
}
},
{
$project: {
_id: 0,
ano: "$ano_cerimonia",
filme: "$nome_do_filme",
categoria: "$categoria"
}
}
])`

**6.2** Quantas indicações a franquia Toy Story recebeu no total?

R: **11 indicações**

`db.filmes.find({"nome_do_filme": /Toy Story/}).count()
`

**6.3** Em quais categorias os filmes Toy Story foram indicados?

`db.filmes.aggregate([
{
$match: {
nome_do_filme: { $regex: "Toy Story", $options: "i" }
}
},
{
$group: {
_id: "$categoria"
}
},
{
$project: {
_id: 0,
categoria_indicada: "$_id"
}
}
])`

### Crash

**6.4** Em qual edição do Oscar o filme "Crash" concorreu?

R: **No ano de 2006**

`db.filmes.find({"nome_do_filme": "Crash"})
`

**6.5** Quantas indicações o filme "Crash" recebeu?

R: **8 indicações**

`db.filmes.find({"nome_do_filme": "Crash"}).count()
`

**6.6** "Crash" ganhou o Oscar de Melhor Filme?

R: **Sim venceu a categoria BEST PICTURE**

`db.filmes.aggregate([
{
$match: {
nome_do_filme: { $regex: "^Crash$", $options: "i" },
categoria: { $in: ["BEST PICTURE", "OUTSTANDING PICTURE"] },
vencedor: { $in: [1, "true"] }
}
},
{
$project: {
_id: 0,
filme: "$nome_do_filme",
ano: "$ano_cerimonia",
categoria_vencida: "$categoria"
}
}
])`

### Central do Brasil

**6.7** O filme "Central do Brasil" aparece no banco de dados?

R: **Sim**

`db.filmes.find({"nome_do_filme": /Central/ })
`

**6.8** Se sim, quantas indicações "Central do Brasil" recebeu?

R: **2 indicações**

`db.filmes.find({"nome_do_filme": /Central/ }).count()
`
---

## 📅 Nível 7: Análise Temporal

**7.1** Quantas indicações aconteceram por década? Agrupe por década (1920s, 1930s, etc.) e mostre o total.

`db.filmes.aggregate([
{
$group: {
_id: {
$multiply: [
{
$floor: {
$divide: [
{ $convert: { input: "$ano_cerimonia", to: "int", onError: 0, onNull: 0 } },
10
]
}
},
10
]
},
total_indicacoes: { $sum: 1 }
}
},
{
$sort: {
_id: 1
}
},
{
$project: {
_id: 0,
decada: { $concat: [ { $toString: "$_id" }, "s" ] },
total_indicacoes: 1
}
}
])
`
**7.2** Em qual década houve o maior número de indicações?

R:   **total_indicacoes: 1501,
decada: '1940s'**

`db.filmes.aggregate([
{
$group: {
_id: {
$multiply: [
{
$floor: {
$divide: [
{ $convert: { input: "$ano_cerimonia", to: "int", onError: 0, onNull: 0 } },
10
]
}
},
10
]
},
total_indicacoes: { $sum: 1 }
}
},
{
$sort: {
total_indicacoes: -1
}
},
{
$limit: 1
},
{
$project: {
_id: 0,
decada: { $concat: [ { $toString: "$_id" }, "s" ] },
total_indicacoes: 1
}
}
])`

**7.3** Como o número de categorias evoluiu ao longo dos anos? Mostre quantas categorias únicas existiam a cada década.

`db.filmes.aggregate([
{
$group: {
_id: {
$multiply: [
{
$floor: {
$divide: [
{ $convert: { input: "$ano_cerimonia", to: "int", onError: 0, onNull: 0 } },
10
]
}
},
10
]
},
categorias_unicas_na_decada: { $addToSet: "$categoria" }
}
},
{
$sort: {
_id: 1
}
},
{
$project: {
_id: 0,
decada: { $concat: [ { $toString: "$_id" }, "s" ] },
total_categorias_unicas: { $size: "$categorias_unicas_na_decada" }
}
}
])`

**7.4** Qual foi o ano com o maior número de indicações registradas?

R: **ano: 1943,
numero_de_indicacoes: 186**

`db.filmes.aggregate([
{
$group: {
_id: "$ano_cerimonia",
total_indicacoes: { $sum: 1 }
}
},
{
$sort: {
total_indicacoes: -1
}
},
{
$limit: 1
},
{
$project: {
_id: 0,
ano: "$_id",
numero_de_indicacoes: "$total_indicacoes"
}
}
])`

**7.5** Calcule a taxa de crescimento de indicações comparando a primeira década com a última.

R:   **primeira_decada: '1920s',
indicacoes_primeira_decada: 73,
ultima_decada: '2020s',
indicacoes_ultima_decada: 722,
taxa_crescimento_percentual: 889.0410958904109**

`db.filmes.aggregate([
{
$group: {
_id: {
$multiply: [
{
$floor: {
$divide: [
{ $convert: { input: "$ano_cerimonia", to: "int", onError: 0, onNull: 0 } },
10
]
}
},
10
]
},
total_indicacoes: { $sum: 1 }
}
},
{
$sort: {
_id: 1
}
},
{
$group: {
_id: null,
dados_decadas: { $push: { decada: "$_id", total: "$total_indicacoes" } }
}
},
{
$project: {
_id: 0,
primeira_decada_dados: { $first: "$dados_decadas" },
ultima_decada_dados: { $last: "$dados_decadas" }
}
},
{
$project: {
primeira_decada: { $concat: [ { $toString: "$primeira_decada_dados.decada" }, "s" ] },
indicacoes_primeira_decada: "$primeira_decada_dados.total",
ultima_decada: { $concat: [ { $toString: "$ultima_decada_dados.decada" }, "s" ] },
indicacoes_ultima_decada: "$ultima_decada_dados.total",
taxa_crescimento_percentual: {
$multiply: [
{
$divide: [
{ $subtract: ["$ultima_decada_dados.total", "$primeira_decada_dados.total"] },
"$primeira_decada_dados.total"
]
},
100
]
}
}
}
])`


---

## 🔄 Nível 8: Operações de Atualização

**8.1** No campo "vencedor", altere todos os valores "true" (string) para true (booleano) e "false" (string) para false (booleano).

`db.filmes.updateMany(
{
vencedor: { $in: ["true", "false"] }
},
[
{
$set: {
vencedor: {
$cond: {
if: { $eq: ["$vencedor", "true"] },
then: true,                         
else: false                        
}
}
}
}
]
)`

**8.2** Inclua no banco 3 filmes que nunca foram nomeados ao Oscar, mas que você acha que merecem. Use dados fictícios mas realistas.

`db.filmes.insertMany([
{
"id_registro": 10990,
"ano_filmagem": 2007,
"ano_cerimonia": 2008,
"cerimonia": 80,
"categoria": "FILM EDITING",
"nome_do_indicado": "Don Zimmerman",
"nome_do_filme": "Rush Hour 3",
"vencedor": false
},
{
"id_registro": 10991,
"ano_filmagem": 2008,
"ano_cerimonia": 2009,
"cerimonia": 81,
"categoria": "BEST PICTURE",
"nome_do_indicado": "Christopher Nolan, Charles Roven and Emma Thomas, Producers",
"nome_do_filme": "The Dark Knight",
"vencedor": false
},
{
"id_registro": 10992,
"ano_filmagem": 1998,
"ano_cerimonia": 1999,
"cerimonia": 71,
"categoria": "ACTOR IN A LEADING ROLE",
"nome_do_indicado": "Jim Carrey",
"nome_do_filme": "The Truman Show",
"vencedor": false
}
])`

**8.3** Corrija possíveis erros de digitação nos nomes dos filmes (ex: espaços extras, caracteres especiais desnecessários).

`db.filmes.updateMany(
{},
[
{
$set: {
nome_do_filme: {
$trim: { input: "$nome_do_filme", chars: " \t\n\r" }
}
}
}
]
)`

**8.5** Remova todos os registros com valor NULL no campo nome_do_filme.

`db.filmes.deleteMany({
nome_do_filme: null
})`

---

## 🎯 Nível 9: Questões Históricas e Sociais

### Representatividade

**9.1** Sidney Poitier foi o primeiro ator negro a ser indicado ao Oscar. Em que ano ele foi indicado? Por qual filme?

R:   **ano_cerimonia: 1959,
nome_do_filme: 'The Defiant Ones',**

`db.filmes.find({"nome_do_indicado": /Sidney Poitier/})
`

**9.2** Sidney Poitier ganhou o Oscar nessa indicação?

R: **Não foi vencedor nessa indicação**

`db.filmes.find({"nome_do_indicado": /Sidney Poitier/})
`


### Coincidências

**9.5** Denzel Washington e Jamie Foxx já concorreram ao Oscar no mesmo ano?

R: **Não concorreram**

`db.filmes.aggregate([
{
$match: {
nome_do_indicado: { $in: ["Denzel Washington", "Jamie Foxx"] }
}
},
{
$group: {
_id: "$ano_cerimonia",
indicados: { $addToSet: "$nome_do_indicado" },
registros: { $push: "$$ROOT" }
}
},
{
$match: {
indicados: { $all: ["Denzel Washington", "Jamie Foxx"] }
}
}
])`


**9.6** Encontre casos onde o mesmo filme ganhou Oscar em múltiplas categorias na mesma cerimônia. Mostre o nome do filme e quantas categorias ele venceu.

`db.filmes.aggregate([
{
$match: {
vencedor: { $in: [1, true] }
}
},
{
$group: {
_id: {
filme: "$nome_do_filme",
ano: "$ano_cerimonia"
},
total_vitorias: { $sum: 1 }
}
},
{
$match: {
total_vitorias: { $gt: 1 }
}
},
{
$sort: {
total_vitorias: -1
}
},
{
$project: {
_id: 0,
filme: "$_id.filme",
ano_da_cerimonia: "$_id.ano",
categorias_vencidas: "$total_vitorias"
}
}
])`
---

## 💡 Dicas Gerais

- **Text Search**: Use regex (`$regex`) para buscar nomes parciais de filmes ou pessoas
- **Conversão de Dados**: Use `$toInt`, `$toBool` para converter tipos
- **Arrays**: Use `$addToSet` para valores únicos
- **Datas**: Use `$year` para extrair o ano de campos de data
- **Performance**: Para queries complexas, considere criar índices em campos frequentemente consultados
- **Boa prática**: Sempre teste com `.limit(5)` antes de executar queries que retornam muitos resultados

---

## 🎓 Avaliação

- **Nível 1-4**: Operações básicas e queries simples
- **Nível 5-8**: Agregações e análise de dados
- **Nível 9-11**: Queries complexas e pensamento analítico
- **Nível 12-14**: Expertise avançada e pensamento estratégico

**Objetivo de aprendizado**: Ao completar todos os níveis, você será capaz de trabalhar com bases de dados históricas complexas, realizar análises estatísticas sofisticadas e extrair insights valiosos de grandes volumes de dados.

Bons estudos e que vença o melhor! 🏆