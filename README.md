Nomeados ao Oscar

Este repositório contém a base de indicados ao Oscar em formato MongoDB, destinada a treinar comandos CRUD e consultas agregadas.

Sobre a base de dados
A base contém informações como:

Nome do indicado
Nome do filme
Categoria
Ano da cerimônia
Se venceu ou não
Ela pode ser utilizada para realizar consultas, atualizações e inserções de registros.



Atividades e Consultas
Pergunta	Comando	Resposta
Atualizar registros da tabela com os dados do Oscar 2025	db.oscar.updateMany({ano_cerimonia: 2024}, {$set: {ano_cerimonia: 2025}})	-
Total de registros na tabela indicados	db.indicados_ao_oscar.countDocuments()	
Número de indicações por categoria	db.indicados_ao_oscar.aggregate([{ $group: { _id: "$categoria", total_indicacoes: { $sum: 1 } } }, { $sort: { total_indicacoes: -1 } }])	2
Quantas vezes Natalie Portman foi indicada	db.oscar.find({"Nome": "Natalie Portman"}).countDocuments()	3
Quantos Oscars Natalie Portman ganhou	db.oscar.find({nome_do_indicado: "Natalie Portman", vencedor: "true"}).count()	1
Quantas vezes Viola Davis foi indicada	db.oscar.find({nome_do_indicado: "Viola Davis"}).count()	4
Quantos Oscars Viola Davis ganhou	db.oscar.find({nome_do_indicado: "Viola Davis", vencedor: "true"}).count()	1
Amy Adams já ganhou algum Oscar?	db.oscar.find({nome_do_indicado: "Amy Adams", vencedor: "true"}).count()	Não
Atores/atrizes com mais de uma indicação	db.oscar.aggregate([{ $match: { categoria: { $in: ["ACTOR", "ACTOR IN A SUPPORTING ROLE", "ACTRESS", "ACTRESS IN A SUPPORTING ROLE"] } } },{ $group: { _id: "$nome_do_indicado", total_indicacoes: { $sum: 1 } } },{ $match: { total_indicacoes: { $gt: 1 } } },{ $sort: { total_indicacoes: -1 } }]).pretty()	-
Anos em que Toy Story ganhou Oscars	db.oscar.aggregate([{ $match: { $and: [ { nome_do_filme: /Toy Story/ }, { vencedor: "true" } ] } },{ $group: { _id: "$ano_cerimonia", titulos_vencedores: { $addToSet: "$nome_do_filme" } } },{ $project: { _id: 0, ano_da_vitoria: "$_id", filmes: "$titulos_vencedores" } },{ $sort: { ano_da_vitoria: 1 } }]).pretty()	2011 e 2020
Categoria "Actress" deixou de existir a partir de	db.oscar.aggregate([{ $match: { categoria: "ACTRESS" } },{ $group: { _id: null, ultimo_ano_encontrado: { $max: "$ano_cerimonia" } } },{ $project: { _id: 0, categoria_verificada: { $literal: "ACTRESS" }, ultimo_ano_de_registro: "$ultimo_ano_encontrado" } }]).pretty()	1977
Primeiro Oscar para Melhor Atriz	db.oscar.aggregate([{ $match: { $and: [ { categoria: "ACTRESS" }, { vencedor: "true" } ] } },{ $sort: { ano_cerimonia: 1 } },{ $limit: 1 },{ $project: { _id: 0, primeira_vencedora: "$nome_do_indicado", ano_da_cerimonia: "$ano_cerimonia", categoria_do_oscar: "$categoria" } }])	Janet Gaynor, 1928
Alterar campo "vencedor" para 1/0	db.oscar.updateMany({vencedor: "true"}, { $set: { vencedor: 1 } }) db.oscar.updateMany({vencedor: "false"}, { $set: { vencedor: 0 } })	-
Edição do Oscar em que "Crash" concorreu	db.oscar.aggregate([{ $match:{ nome_do_filme: "Crash" } },{ $group: { _id:{ ano: "$ano_cerimonia", edicao: "$cerimonia" } } },{ $project: { _id: 0, ano_da_cerimonia: "$_id.ano", edicao_do_oscar: "_id.edicao" } }]).pretty()	2006
Central do Brasil aparece no Oscar	db.oscar.find({ nome_do_filme: /Central Station/ }).projection({ _id:0, categoria: 1, nome_do_indicado: 1, vencedor: 1, ano_cerimonia: 1 }).pretty()	1999
Inserir 3 filmes nunca indicados	db.oscar.insertMany([{ "id_registro": 10890, "ano_filmagem": 1993, "ano_cerimonia": 1994, "cerimonia": 64, "categoria": "ACTOR", "nome_do_indicado":"Bill Murray", "nome_do_filme": "Feitiço do Tempo" },{ "id_registro": 10891, "ano_filmagem": 1999, "ano_cerimonia": 2000, "cerimonia": 72, "categoria": "ACTOR", "nome_do_indicado":"Jeff Bridges", "nome_do_filme": "O Grande Lebowski" },{ "id_registro": 10892, "ano_filmagem": 2003, "ano_cerimonia": 2004, "cerimonia": 76, "categoria": "ACTOR", "nome_do_indicado":"Jake Gyllenhaal", "nome_do_filme": "Donnie Darko" }])	-
Denzel Washington já ganhou algum Oscar?	db.oscar.find({nome_do_indicado: "Denzel Washington", vencedor: 1}).count()	2
Filmes que ganharam Melhor Filme	db.oscar.aggregate([{ $match: { $and: [ { categoria: "BEST MOTION PICTURE" }, { vencedor: 1 } ] } },{ $project: { _id: 0, nome_do_filme: "$nome_do_filme", ano_da_cerimonia: "$ano_cerimonia", categoria_do_oscar: "$categoria" } }])	SIM
Sidney Poitier - primeira indicação	db.oscar.aggregate([{ $match:{nome_do_indicado: "Sidney Poitier"} },{ $project: { _id: 0, nome_do_filme: "$nome_do_filme", ano_da_cerimonia: "$ano_cerimonia", categoria_do_oscar: "$categoria" } }])	1959, The Defiant Ones
Filmes que ganharam Melhor Filme e Diretor na mesma cerimônia	db.oscar.aggregate([{ $match: { vencedor: 1, categoria: { $in: ["BEST MOTION PICTURE", "DIRECTING"] } } },{ $group: { _id: { cerimonia: "$cerimonia", ano: "$ano_cerimonia", filme: "$nome_do_filme" }, categorias_vencidas: { $addToSet: "$categoria" } } },{ $match: { categorias_vencidas: { $all: ["BEST MOTION PICTURE", "DIRECTING"] } } },{ $project: { _id: 0, filme: "$_id.filme", edicao_da_cerimonia: "$_id.cerimonia", ano_da_cerimonia: "$_id.ano" } },{ $sort: { ano_da_cerimonia: 1 } }])	SIM
Denzel Washington e Jamie Foxx no mesmo ano?	db.oscar.aggregate([{ $match: { nome_do_indicado:{ $in: ["Denzel Washington", "Jamie Foxx"] } } },{ $group: { _id: "$ano_cerimonia", indicados_no_ano: { $addToSet: "$nome_do_indicado"} } },{ $match: { indicados_no_ano: { $size: 2 } } },{ $project:{ _id: 0, ano_da_cerimonia: "$_id", indicados: "$indicados_no_ano" } },{ $sort: { ano_da_cerimonia: 1 } }])	NÃO
