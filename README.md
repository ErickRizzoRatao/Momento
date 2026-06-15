
# --- O momento. 

"Se você pudesse voltar, você voltaria?"
Inspirados pela narrativa e pela estética temporal de The Moment (Charli XCX), a momento é uma startup de tecnologia focada em infraestrutura de dados hiper-responsiva, compressão de memória analógica e interfaces que desafiam a linearidade do tempo digital.


Estrutura do Banco de Dados
Para sustentar a operação, a "Momento" utiliza um sistema de banco de dados não relacional que integra os colaboradores aos seus respectivos departamentos.

## Nível 1: Conhecendo a empresa.

**1.1** Inclua suas próprias informações no departamento de Tecnologia da empresa.
R: 
```bson
insertOne({
  "nome": "Erick",
  "sobrenome": "Rizzo",
  "data_nascimento": "2006-05-12",
  "data_contratacao": "2026-05-10",
  "salario": 8000,
  "departamento": "Tecnologia",
  "cargo": "Desenvolvedor",
  "escritorio": "São Paulo"
})
```
**1.2** Agora que você faz parte da equipe, quantos funcionários temos ao total na empresa?
R: 41 Funcionários. 

**1.3** Quantos funcionarios trabalham especificamente no Departamento de Tecnologia?

R: 9 funcionarios.
```bson
db.funcionarios.countDocuments({ departamento: "Tecnologia" })
```

**1.4** Liste todos os departamentos que existem na empresa. Quantos sao?
R: 19 departamentos.
```bson
db.funcionarios.distinct("departamento")
```

**Resultado**
Compras, Cozinha e Refeitorio, Dados, Design, Diretoria, Financeiro, Infraestrutura, Juridico, Limpeza e Conservacao, Logistica, Manutencao, Marketing, Produto, RH, Recepcao, Seguranca, Seguranca Patrimonial, Suporte e Tecnologia.

**1.5** Quantos escritorios a Momento possui? Em quais paises?
R: Com os arquivos disponiveis, ha 1 escritorio: Sao Paulo. O pais nao esta salvo no JSON de funcionarios; considerando o escritorio, e Brasil.
```bson
db.funcionarios.distinct("escritorio")
```

---

## Nível 2: Análise Financeira Básica

**2.1** Quantos funcionarios trabalham no Departamento de Vendas?
R: 0 funcionarios.
```bson
db.funcionarios.countDocuments({ departamento: "Vendas" })
```

**2.2** Qual e o custo total com salarios do Departamento de Vendas?
R: 0.
```bson
db.funcionarios.aggregate([
  { $match: { departamento: "Vendas" } },
  { $group: { _id: null, custo_total: { $sum: "$salario" } } }
])
```

**2.3** Qual e a media salarial da empresa, excluindo os cargos de CEO, CMO e CFO?
R: 6076.92.
```bson
db.funcionarios.aggregate([
  { $match: { cargo: { $nin: ["CEO", "CMO", "CFO"] } } },
  { $group: { _id: null, mediaSalarial: { $avg: "$salario" } } },
  { $project: { _id: 0, mediaSalarial: { $round: ["$mediaSalarial", 2] } } }
])
```

**2.4** Qual e a media salarial do Departamento de Tecnologia?
R: 9222.22.
```bson
db.funcionarios.aggregate([
  { $match: { departamento: "Tecnologia" } },
  { $group: { _id: null, mediaSalarial: { $avg: "$salario" } } },
  { $project: { _id: 0, mediaSalarial: { $round: ["$mediaSalarial", 2] } } }
])
```

**2.5** Qual departamento possui a maior media salarial?
R: Diretoria, com media salarial de 25000.
```bson
db.funcionarios.aggregate([
  { $group: { _id: "$departamento", mediaSalarial: { $avg: "$salario" } } },
  { $sort: { mediaSalarial: -1 } },
  { $limit: 1 }
])
```

**2.6** Qual departamento possui o menor numero de funcionarios?
R: Ha empate com 1 funcionario: Compras, Diretoria, Infraestrutura, Juridico, Marketing, RH, Seguranca e Suporte.
```bson
db.funcionarios.aggregate([
  { $group: { _id: "$departamento", totalFuncionarios: { $sum: 1 } } },
  { $sort: { totalFuncionarios: 1, _id: 1 } }
])
```

---

## Nível 3: Recursos Humanos

O RH está fazendo uma análise demográfica da empresa.

**3.1** Quantos funcionarios da empresa Momento possuem conjuges?
R: 0 nos arquivos disponiveis, porque os funcionarios exportados nao possuem campo `dependentes`.
```bson
db.funcionarios.countDocuments({
  "dependentes.conjuge": { $exists: true, $ne: null }
})
```

**3.2** Quantos funcionarios possuem filhos registrados?
R: 0 nos arquivos disponiveis, porque os funcionarios exportados nao possuem campo `dependentes`.
```bson
db.funcionarios.countDocuments({
  "dependentes.filhos.0": { $exists: true }
})
```

**3.3** Qual funcionário foi contratado há mais tempo na empresa?
R:  Sebastião Ferreira
```bson
db.funcionarios.findOne(
  {},
  { nome: 1, sobrenome: 1, data_contratacao: 1, _id: 0 },
  { sort: { data_contratacao: 1 } }
)
```

**3.4** Qual funcionário foi contratado há menos tempo na empresa?
R: Erick Rizzo
```bson
db.funcionarios.findOne(
  {},
  { nome: 1, sobrenome: 1, data_contratacao: 1, _id: 0 },
  { sort: { data_contratacao: -1 } }
)
```

**3.5** Liste os 5 funcionários com mais tempo de casa, ordenados pela data de contratação.
R: 
```bson
db.funcionarios.find(
  {},
  { nome: 1, sobrenome: 1, cargo: 1, data_contratacao: 1, _id: 0 }
).sort({ data_contratacao: 1 }).limit(5)
```

**3.6** Quantos funcionarios foram contratados na decada de 1990 (entre 1990-1999)?
R: 0 funcionarios.
```bson
db.funcionarios.countDocuments({
  data_contratacao: {
    $gte: "1990-01-01",
    $lte: "1999-12-31"
  }
})
```

**3.7** Como a média salarial da Momento evoluiu ao longo dos anos? Agrupe por ano de contratação e calcule a média salarial.
R: 
```bson
db.funcionarios.aggregate([
  {
    $group: {
      _id: { $substr: ["$data_contratacao", 0, 4] },
      media_salarial: { $avg: "$salario" },
      total_funcionarios: { $sum: 1 }
    }
  },
  { $sort: { _id: 1 } },
  {
    $project: {
      _id: 0,
      ano: "$_id",
      media_salarial: { $round: ["$media_salarial", 2] },
      total_funcionarios: 1
    }
  }
])
```
---

## Nível 4: Operações e Escritórios

**4.1** Qual é o custo total de suprimentos em cada escritório? Ordene do mais caro ao mais barato.
R: 
```bson
db.suprimentos.aggregate([
  {
    $group: {
      _id: "$escritorio",
      custo_total: { $sum: { $multiply: ["$quantidade", "$preco_unitario"] } }
    }
  },
  { $sort: { custo_total: -1 } },
  {
    $project: {
      _id: 0,
      escritorio: "$_id",
      custo_total: { $round: ["$custo_total", 2] }
    }
  }
])
```
**4.2** Qual escritório possui a maior quantidade de diferentes tipos de suprimentos?
R: 
```bson
db.suprimentos.aggregate([
  {
    $group: {
      _id: "$escritorio",
      tipos_distintos: { $addToSet: "$nome" }
    }
  },
  {
    $project: {
      _id: 0,
      escritorio: "$_id",
      quantidade_tipos: { $size: "$tipos_distintos" }
    }
  },
  { $sort: { quantidade_tipos: -1 } },
  { $limit: 1 }
])

```

**4.3** Qual é o suprimento mais caro (considerando preço unitário) em toda a empresa?
R: 
```bson
db.suprimentos.findOne(
  {},
  { nome: 1, escritorio: 1, preco_unitario: 1, _id: 0 },
  { sort: { preco_unitario: -1 } }
)
```

**4.4** Calcule o valor total do inventário de suprimentos da empresa (quantidade × preço unitário de todos os itens em todos os escritórios).
R: 
```bson
db.suprimentos.aggregate([
  {
    $group: {
      _id: null,
      valor_total_inventario: {
        $sum: { $multiply: ["$quantidade", "$preco_unitario"] }
      }
    }
  },
  {
    $project: {
      _id: 0,
      valor_total_inventario: { $round: ["$valor_total_inventario", 2] }
    }
  }
])
```
---

## Nível 5: Produtos e Vendas

**5.1** Quais produtos foram vendidos pela Momento? Liste todos os produtos únicos.
R: ```bson
db.vendas.distinct("produto")
```


**5.2** Qual é o produto mais vendido (maior quantidade total)?
R:
```bson
db.vendas.aggregate([
  { $group: { _id: "$produto", quantidade_total: { $sum: "$quantidade" } } },
  { $sort: { quantidade_total: -1 } },
  { $limit: 1 },
  { $project: { _id: 0, produto: "$_id", quantidade_total: 1 } }
])
```

**5.3** Qual é o produto menos vendido? R: 
R: ```bson
db.vendas.aggregate([
  { $group: { _id: "$produto", quantidade_total: { $sum: "$quantidade" } } },
  { $sort: { quantidade_total: 1 } },
  { $limit: 1 },
  { $project: { _id: 0, produto: "$_id", quantidade_total: 1 } }
])
```

**5.4** Pensando na relação quantidade × valor unitário, qual produto gerou mais receita para a empresa?
R:
```bson
db.vendas.aggregate([
  {
    $group: {
      _id: "$produto",
      receita_total: { $sum: "$valor_total" }
    }
  },
  { $sort: { receita_total: -1 } },
  { $limit: 1 },
  {
    $project: {
      _id: 0,
      produto: "$_id",
      receita_total: { $round: ["$receita_total", 2] }
    }
  }
])
```

**5.5** Qual é o produto mais caro (maior preço unitário) no catálogo?
R: 
```bson
db.vendas.findOne(
  {},
  { produto: 1, preco_unitario: 1, _id: 0 },
  { sort: { preco_unitario: -1 } }
)
```

**5.6** Qual foi o faturamento total da empresa em vendas?
R: 
```bson
db.vendas.aggregate([
  { $match: { status: "Concluída" } },
  {
    $group: {
      _id: null,
      faturamento_total: { $sum: "$valor_total" }
    }
  },
  {
    $project: {
      _id: 0,
      faturamento_total: { $round: ["$faturamento_total", 2] }
    }
  }
])
```
**5.7** Quantas vendas foram realizadas no mês de junho de 2023?
R: ```bson
db.vendas.countDocuments({
  data_venda: { $gte: "2023-06-01", $lte: "2023-06-30" }
})
```

**5.8** Qual vendedor realizou mais vendas (em quantidade de transações)?
R: ```bson
 db.vendas.aggregate([
  { $match: { status: "Concluída" } },
  { $group: { _id: "$vendedor", total_vendas: { $sum: 1 } } },
  { $sort: { total_vendas: -1 } },
  { $limit: 1 },
  { $project: { _id: 0, vendedor: "$_id", total_vendas: 1 } }
])
```

**5.9** Qual vendedor gerou mais receita para a empresa?
R: ```bson
db.vendas.aggregate([
  { $match: { status: "Concluída" } },
  {
    $group: {
      _id: "$vendedor",
      receita_total: { $sum: "$valor_total" }
    }
  },
  { $sort: { receita_total: -1 } },
  { $limit: 1 },
  {
    $project: {
      _id: 0,
      vendedor: "$_id",
      receita_total: { $round: ["$receita_total", 2] }
    }
  }
])
```

---

## Nível 6: Operações de Atualização

A empresa está crescendo e mudanças precisam acontecer!

**6.1** Um novo departamento foi criado: **Inovações**. Ele será alocado no escritório "Wayne Offices". Adicione-o ao banco de dados.
R:
```bson
db.escritorios.insertOne({
  nome: "Wayne Offices",
  tipo: "Filial",
  status: "Ativo",
  departamentos: ["Inovações"],
  data_inauguracao: new Date().toISOString().split("T")[0]
})
```

**6.2** O departamento de Inovações está sem funcionários. Transfira 2 funcionários do departamento de Tecnologia para Inovações.
R: 
```bson
// Primeiro veja quem está disponível
db.funcionarios.find(
  { departamento: "Tecnologia" },
  { nome: 1, sobrenome: 1, cargo: 1 }
).limit(5)

// Depois transfira pelos _id escolhidos
db.funcionarios.updateMany(
  { _id: { $in: [ObjectId("..."), ObjectId("...")] } },
  { $set: { departamento: "Inovações", escritorio: "Wayne Offices" } }
)
```
**6.3** A empresa decidiu dar um aumento de 10% para todos os funcionários do departamento de Tecnologia. Atualize os salários.
R: 
```bson
db.funcionarios.updateMany(
  { departamento: "Tecnologia" },
  { $mul: { salario: 1.10 } }
)
```

**6.4** O funcionário "Bruce Ernst" foi promovido a "Senior Web Developer" e recebeu um aumento para $5.000. Atualize suas informações.
R:
```bson
db.funcionarios.updateOne(
  { nome: "Bruce", sobrenome: "Ernst" },
  {
    $set: {
      cargo: "Senior Web Developer",
      salario: 5000
    }
  }
)
```
**6.5** Adicione um novo suprimento ao escritório "Wayne Offices": 15 unidades de "Headsets" com preço unitário de $150 cada.
R: 
```bson
db.suprimentos.insertOne({
  escritorio: "Wayne Offices",
  nome: "Headsets",
  categoria: "Informática",
  quantidade: 15,
  preco_unitario: 150.00,
  ultima_reposicao: new Date().toISOString().split("T")[0]
})
```

**6.6** Todos os funcionários contratados antes de 1990 estão aposentando. Remova-os do banco de dados (CUIDADO: execute um **find** antes para ver quantos serão afetados!).
R: 
```bson
db.funcionarios.find(
  { data_contratacao: { $lt: "1990-01-01" } },
  { nome: 1, sobrenome: 1, data_contratacao: 1, cargo: 1 }
)

// Só depois, se estiver tudo certo, execute o delete:
db.funcionarios.deleteMany(
  { data_contratacao: { $lt: "1990-01-01" } }
)
```

---

---

## Nivel 7: Analise Avancada com Agregacoes

**7.1** Crie um relatorio mostrando cada departamento com: nome do departamento, numero de funcionarios, salario total e media salarial.
R:
~~~bson
db.funcionarios.aggregate([
  { $group: { _id: "$departamento", total_funcionarios: { $sum: 1 }, salario_total: { $sum: "$salario" }, media_salarial: { $avg: "$salario" } } },
  { $project: { _id: 0, departamento: "$_id", total_funcionarios: 1, salario_total: 1, media_salarial: { $round: ["$media_salarial", 2] } } },
  { $sort: { departamento: 1 } }
])
~~~

**7.2** Liste os 3 cargos mais comuns na empresa e quantos funcionarios ocupam cada cargo.
R:
~~~bson
db.funcionarios.aggregate([
  { $group: { _id: "$cargo", total: { $sum: 1 } } },
  { $sort: { total: -1, _id: 1 } },
  { $limit: 3 },
  { $project: { _id: 0, cargo: "$_id", total: 1 } }
])
~~~

**7.3** Encontre todos os funcionarios que ganham acima da media salarial de seus respectivos departamentos.
R:
~~~bson
db.funcionarios.aggregate([
  { $setWindowFields: { partitionBy: "$departamento", output: { media_departamento: { $avg: "$salario" } } } },
  { $match: { $expr: { $gt: ["$salario", "$media_departamento"] } } },
  { $project: { _id: 0, nome: 1, sobrenome: 1, departamento: 1, cargo: 1, salario: 1, media_departamento: { $round: ["$media_departamento", 2] } } },
  { $sort: { departamento: 1, salario: -1 } }
])
~~~

**7.4** Calcule a taxa de crescimento da empresa por ano.
R:
~~~bson
db.funcionarios.aggregate([
  { $group: { _id: { $substr: ["$data_contratacao", 0, 4] }, contratados: { $sum: 1 } } },
  { $project: { _id: 0, ano: "$_id", contratados: 1 } },
  { $sort: { ano: 1 } }
])
~~~

**7.5** Crie um ranking dos vendedores.
R:
~~~bson
db.vendas.aggregate([
  { $match: { vendedor: { $exists: true, $ne: null } } },
  { $group: { _id: "$vendedor", total_vendas: { $sum: 1 }, receita_total: { $sum: { $multiply: ["$quantidade", "$preco_unitario"] } } } },
  { $sort: { receita_total: -1, total_vendas: -1 } },
  { $project: { _id: 0, vendedor: "$_id", total_vendas: 1, receita_total: 1 } }
])
~~~

**7.6** Encontre os produtos que foram vendidos por apenas um vendedor.
R:
~~~bson
db.vendas.aggregate([
  { $group: { _id: "$produto", vendedores: { $addToSet: "$vendedor" } } },
  { $match: { $expr: { $eq: [{ $size: "$vendedores" }, 1] } } },
  { $project: { _id: 0, produto: "$_id", vendedor: { $first: "$vendedores" } } }
])
~~~

---

## Nivel 8: Desafios e Otimizacao

**8.1** Encontre todos os funcionarios do departamento de Vendas que possuem dependentes.
R:
~~~bson
// Forma 1
db.funcionarios.find({ departamento: "Vendas", $or: [{ "dependentes.conjuge": { $exists: true, $ne: null } }, { "dependentes.filhos.0": { $exists: true } }] })

// Forma 2
db.funcionarios.find({ departamento: "Vendas", $expr: { $or: [{ $ne: ["$dependentes.conjuge", null] }, { $gt: [{ $size: { $ifNull: ["$dependentes.filhos", []] } }, 0] }] } })
~~~

**8.2** Encontre funcionarios que trabalham em escritorios diferentes do escritorio associado ao departamento.
R:
~~~bson
db.funcionarios.aggregate([
  { $lookup: { from: "departamentos", localField: "departamento", foreignField: "nome", as: "departamento_info" } },
  { $unwind: "$departamento_info" },
  { $match: { $expr: { $ne: ["$escritorio", "$departamento_info.escritorio"] } } },
  { $project: { _id: 0, nome: 1, sobrenome: 1, departamento: 1, escritorio_funcionario: "$escritorio", escritorio_departamento: "$departamento_info.escritorio" } }
])
~~~

**8.3** Crie um relatorio completo de um escritorio.
R:
~~~bson
db.escritorios.aggregate([
  { $match: { nome: "Sao Paulo" } },
  { $lookup: { from: "departamentos", localField: "nome", foreignField: "escritorio", as: "departamentos" } },
  { $lookup: { from: "funcionarios", localField: "nome", foreignField: "escritorio", as: "funcionarios" } },
  { $lookup: { from: "suprimentos", localField: "nome", foreignField: "escritorio", as: "suprimentos" } },
  { $project: { _id: 0, escritorio: "$nome", pais: 1, numero_departamentos: { $size: "$departamentos" }, numero_funcionarios: { $size: "$funcionarios" }, custo_salarios: { $sum: "$funcionarios.salario" }, custo_suprimentos: { $sum: { $map: { input: "$suprimentos", as: "item", in: { $multiply: ["$$item.quantidade", "$$item.preco_unitario"] } } } } } }
])
~~~

**8.4** Encontre o departamento mais equilibrado.
R:
~~~bson
db.funcionarios.aggregate([
  { $group: { _id: "$departamento", maior_salario: { $max: "$salario" }, menor_salario: { $min: "$salario" }, total_funcionarios: { $sum: 1 } } },
  { $match: { total_funcionarios: { $gt: 1 } } },
  { $addFields: { diferenca: { $subtract: ["$maior_salario", "$menor_salario"] } } },
  { $sort: { diferenca: 1 } },
  { $limit: 1 },
  { $project: { _id: 0, departamento: "$_id", maior_salario: 1, menor_salario: 1, diferenca: 1 } }
])
~~~

**8.5** Text Search: encontre todos os produtos que contem a palavra "Uniforme" no nome.
R:
~~~bson
db.produtos.find({ nome: /Uniforme/i })
~~~

**8.6** Date Range: encontre todas as vendas que ocorreram no segundo trimestre de 2023.
R:
~~~bson
db.vendas.find({ data_venda: { $gte: "2023-04-01", $lte: "2023-06-30" } })
~~~

---

## Nivel 9: Desafios Ninja

**9.1** Encontre todos os funcionarios cujo salario esta entre 6000 e 10000. Resolva de 3 formas diferentes.
R:
~~~bson
db.funcionarios.find({ salario: { $gte: 6000, $lte: 10000 } })
db.funcionarios.find({ $and: [{ salario: { $gte: 6000 } }, { salario: { $lte: 10000 } }] })
db.funcionarios.aggregate([{ $match: { salario: { $gte: 6000, $lte: 10000 } } }])
~~~

**9.2** Qual query e mais eficiente para encontrar funcionarios de Vendas com salario acima de 7000?
R: A opcao A e mais eficiente, porque o filtro roda no MongoDB e pode usar indice composto. A opcao B filtra parte dos dados no cliente.
~~~bson
db.funcionarios.createIndex({ departamento: 1, salario: 1 })
db.funcionarios.find({ departamento: ObjectId("..."), salario: { $gt: 7000 } })
~~~

**9.3** Encontre todos os funcionarios que NAO possuem email ou telefone cadastrado.
R:
~~~bson
db.funcionarios.find({ $or: [{ email: { $exists: false } }, { email: null }, { telefone: { $exists: false } }, { telefone: null }] })
~~~

**9.4** Crie uma query que encontre funcionarios solitarios.
R:
~~~bson
db.funcionarios.aggregate([
  { $group: { _id: { departamento: "$departamento", cargo: "$cargo" }, total: { $sum: 1 }, funcionarios: { $push: { nome: "$nome", sobrenome: "$sobrenome" } } } },
  { $match: { total: 1 } },
  { $unwind: "$funcionarios" },
  { $project: { _id: 0, departamento: "$_id.departamento", cargo: "$_id.cargo", nome: "$funcionarios.nome", sobrenome: "$funcionarios.sobrenome" } }
])
~~~

**9.5** Pipeline complexo: relatorio por pais.
R:
~~~bson
db.escritorios.aggregate([
  { $lookup: { from: "departamentos", localField: "nome", foreignField: "escritorio", as: "departamentos" } },
  { $lookup: { from: "funcionarios", localField: "nome", foreignField: "escritorio", as: "funcionarios" } },
  { $lookup: { from: "suprimentos", localField: "nome", foreignField: "escritorio", as: "suprimentos" } },
  { $group: { _id: "$pais", escritorios: { $addToSet: "$nome" }, departamentos: { $sum: { $size: "$departamentos" } }, funcionarios: { $sum: { $size: "$funcionarios" } }, salarios: { $sum: { $sum: "$funcionarios.salario" } }, suprimentos: { $sum: { $sum: { $map: { input: "$suprimentos", as: "item", in: { $multiply: ["$$item.quantidade", "$$item.preco_unitario"] } } } } } } },
  { $project: { _id: 0, pais: "$_id", numero_escritorios: { $size: "$escritorios" }, numero_departamentos: "$departamentos", numero_funcionarios: "$funcionarios", receita_total: { $add: ["$salarios", "$suprimentos"] } } }
])
~~~

---

## Nivel 10: Casos Praticos e Simulacoes

**10.1** Crie o novo escritorio "Momento Brasil" em Sao Paulo com suprimentos iniciais.
R:
~~~bson
db.escritorios.insertOne({ nome: "Momento Brasil", cidade: "Sao Paulo", pais: "Brasil", data_abertura: "2026-06-07" })
db.suprimentos.insertMany([
  { escritorio: "Momento Brasil", nome: "Notebooks", quantidade: 10, preco_unitario: 3500 },
  { escritorio: "Momento Brasil", nome: "Cadeiras Ergonomicas", quantidade: 10, preco_unitario: 850 },
  { escritorio: "Momento Brasil", nome: "Headsets", quantidade: 15, preco_unitario: 150 }
])
~~~

**10.2** Crie um novo departamento "Operacoes LATAM" vinculado a este escritorio.
R:
~~~bson
db.departamentos.insertOne({ nome: "Operacoes LATAM", escritorio: "Momento Brasil", area: "Operacoes" })
~~~

**10.3** Contrate 5 novos funcionarios para este departamento.
R:
~~~bson
db.funcionarios.insertMany([
  { nome: "Erick", sobrenome: "Rizzo", data_nascimento: "2007-12-20", data_contratacao: "2026-06-07", salario: 8000, departamento: "Operacoes LATAM", cargo: "Analista de Operacoes LATAM", escritorio: "Momento Brasil" },
  { nome: "Marina", sobrenome: "Almeida", data_nascimento: "2001-03-12", data_contratacao: "2026-06-07", salario: 6200, departamento: "Operacoes LATAM", cargo: "Assistente de Operacoes", escritorio: "Momento Brasil" },
  { nome: "Joao", sobrenome: "Pereira", data_nascimento: "2000-08-19", data_contratacao: "2026-06-07", salario: 6200, departamento: "Operacoes LATAM", cargo: "Assistente de Operacoes", escritorio: "Momento Brasil" },
  { nome: "Beatriz", sobrenome: "Santos", data_nascimento: "2002-11-04", data_contratacao: "2026-06-07", salario: 6500, departamento: "Operacoes LATAM", cargo: "Analista de Relacionamento", escritorio: "Momento Brasil" },
  { nome: "Rafael", sobrenome: "Costa", data_nascimento: "1999-05-28", data_contratacao: "2026-06-07", salario: 7000, departamento: "Operacoes LATAM", cargo: "Coordenador LATAM", escritorio: "Momento Brasil" }
])
~~~

**10.4** Identifique os 3 escritorios com maiores custos de suprimentos.
R:
~~~bson
db.suprimentos.aggregate([
  { $group: { _id: "$escritorio", custo_total: { $sum: { $multiply: ["$quantidade", "$preco_unitario"] } } } },
  { $sort: { custo_total: -1 } },
  { $limit: 3 },
  { $project: { _id: 0, escritorio: "$_id", custo_total: 1 } }
])
~~~

**10.5** Sugira onde cortar custos: liste suprimentos com quantidade acima de 50 unidades.
R:
~~~bson
db.suprimentos.find({ quantidade: { $gt: 50 } }, { _id: 0, escritorio: 1, nome: 1, quantidade: 1, preco_unitario: 1 }).sort({ quantidade: -1 })
~~~

**10.6** Calcule quanto a empresa economizaria se reduzisse 20% dos salarios acima de 15000.
R:
~~~bson
db.funcionarios.aggregate([
  { $match: { salario: { $gt: 15000 } } },
  { $group: { _id: null, economia: { $sum: { $multiply: ["$salario", 0.20] } }, funcionarios_afetados: { $sum: 1 } } },
  { $project: { _id: 0, economia: 1, funcionarios_afetados: 1 } }
])
~~~

**10.7** Encontre vendas que nao possuem vendedor associado.
R:
~~~bson
db.vendas.find({ $or: [{ vendedor: { $exists: false } }, { vendedor: null }, { vendedor: "" }] })
~~~

**10.8** Verifique se ha vendedores registrados nas vendas que nao existem na colecao de funcionarios.
R:
~~~bson
db.vendas.aggregate([
  { $lookup: { from: "funcionarios", localField: "vendedor", foreignField: "_id", as: "funcionario" } },
  { $match: { vendedor: { $exists: true, $ne: null }, funcionario: { $size: 0 } } },
  { $project: { _id: 1, vendedor: 1, produto: 1, data_venda: 1 } }
])
~~~

**10.9** Identifique produtos com precos unitarios muito diferentes entre transacoes.
R:
~~~bson
db.vendas.aggregate([
  { $group: { _id: "$produto", menor_preco: { $min: "$preco_unitario" }, maior_preco: { $max: "$preco_unitario" } } },
  { $addFields: { variacao_percentual: { $multiply: [{ $divide: [{ $subtract: ["$maior_preco", "$menor_preco"] }, "$menor_preco"] }, 100] } } },
  { $match: { variacao_percentual: { $gt: 10 } } },
  { $sort: { variacao_percentual: -1 } }
])
~~~

**10.10** Crie uma unica query de agregacao que retorne o dashboard completo.
R:
~~~bson
db.funcionarios.aggregate([
  {
    $facet: {
      funcionarios: [
        { $group: { _id: null, total_funcionarios: { $sum: 1 }, custo_total_salarios: { $sum: "$salario" }, departamentos: { $addToSet: "$departamento" }, escritorios: { $addToSet: "$escritorio" } } }
      ],
      vendas: [
        { $lookup: { from: "vendas", pipeline: [{ $group: { _id: null, receita_total_vendas: { $sum: { $multiply: ["$quantidade", "$preco_unitario"] } } } }], as: "dados" } },
        { $limit: 1 }
      ],
      produto_mais_vendido: [
        { $lookup: { from: "vendas", pipeline: [{ $group: { _id: "$produto", quantidade_total: { $sum: "$quantidade" } } }, { $sort: { quantidade_total: -1 } }, { $limit: 1 }], as: "produto" } },
        { $limit: 1 }
      ]
    }
  },
  { $project: { _id: 0, total_funcionarios: { $first: "$funcionarios.total_funcionarios" }, custo_total_salarios: { $first: "$funcionarios.custo_total_salarios" }, numero_departamentos: { $size: { $first: "$funcionarios.departamentos" } }, numero_escritorios: { $size: { $first: "$funcionarios.escritorios" } }, receita_total_vendas: { $first: "$vendas.dados.receita_total_vendas" }, produto_mais_vendido: { $first: "$produto_mais_vendido.produto" } } }
])
~~~
