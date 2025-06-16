# Banco de Dados de um Hospital

# Parte 1 - Mãos a Obra 
  
Analise a seguinte descrição e extraia dela os requisitos para o banco de dados em um diagrama, fluxograma ou afins:
O hospital necessita de um sistema para sua área clínica que ajude a controlar consultas realizadas. Os médicos podem ser generalistas, especialistas ou residentes e têm seus dados pessoais cadastrados em planilhas digitais. Cada médico pode ter uma ou mais especialidades, que podem ser pediatria, clínica geral, gastroenterologia e dermatologia. Alguns registros antigos ainda estão em formulário de papel, mas será necessário incluir esses dados no novo sistema.

<img src="images/medicoBase.png" alt="Base de Dados Medicos" width="550"/>

Os pacientes também precisam de cadastro, contendo dados pessoais (nome, data de nascimento, endereço, telefone e e-mail), documentos (CPF e RG) e convênio. Para cada convênio, são registrados nome, CNPJ e tempo de carência.

<img src="images/pacientesBase.png" alt="Base de Dados Paciente" width="550"/>

As consultas também têm sido registradas em planilhas, com data e hora de realização, médico responsável, paciente, valor da consulta ou nome do convênio, com o número da carteira. Também é necessário indicar na consulta qual a especialidade buscada pelo paciente.

Deseja-se ainda informatizar a receita do médico, de maneira que, no encerramento da consulta, ele possa registrar os medicamentos receitados, a quantidade e as instruções de uso. A partir disso, espera-se que o sistema imprima um relatório da receita ao paciente ou permita sua visualização via internet.

<img src="images/consultasBase.png" alt="Base de Dados Consultas" width="450"/>

# Parte 2 - Não era exatamente assim 

Considere a seguinte descrição:

No hospital, as internações têm sido registradas por meio de formulários eletrônicos que gravam os dados em arquivos. 

Para cada internação, são anotadas a data de entrada, a data prevista de alta e a data efetiva de alta, além da descrição textual dos procedimentos a serem realizados. 

**Obs: colocadas junto com paciente, com a caracteristica em boolean se esta internado ou não**

As internações precisam ser vinculadas a quartos, com a numeração e o tipo. 

Cada tipo de quarto tem sua descrição e o seu valor diário (a princípio, o hospital trabalha com apartamentos, quartos duplos e enfermaria).

<img src="images/quartosBase.png" alt="Base de Dados Quartos" width="250">

Também é necessário controlar quais profissionais de enfermaria estarão responsáveis por acompanhar o paciente durante sua internação. Para cada enfermeiro(a), é necessário nome, CPF e registro no conselho de enfermagem (COREN).
A internação, obviamente, é vinculada a um paciente – que pode se internar mais de uma vez no hospital – e a um único médico responsável.

<img src="images/enfermeirosBase.png" alt="Base de Dados Enfermeiros" width="450">

# Parte 3  - Jogando nas regras que você criou:  
Crie scripts de povoamento dos documentos desenvolvidas na atividade anterior
Observe as seguintes atividades: 

- 12 medicos, ao menos sete especialidades (considere a afirmação de que “entre as especialidades há pediatria, clínica geral, gastrenterologia e dermatologia”).
- 15 pacientes
- 20 consultas
- Relacione as internações com IDs de Médicos e Pacientes
- 7 internações (entre 01/01/2015 e 01/01/2022)
- 10 profissionais de enfermaria

Parte 4 - Inserido Dados 
Pensando no banco que já foi criado para o Projeto do Hospital, realize algumas alterações nas tabelas e nos dados usando comandos de atualização e exclusão:

Crie um script que adicione uma coluna “em_atividade” para os médicos, indicando se ele ainda está atuando no hospital ou não.

Query:

```js
db.medicos.updateMany( {}, { $set: em_atividade: true } )
```

Crie um script para atualizar ao menos dois médicos como inativos e os demais em atividade.

Query:

```js
db.medicos.updateMany( { nome_medico: { $in: [ "Dra. Mariana Souza", "Dr. Gustavo Carvalho" ] } }, { $set: { em_atividade: false } } )
```
# Parte 5 - Consultas para que te quero 

Crie um script e nele inclua consultas que retornem:

1. Todos os dados e o valor médio das consultas do ano de 2020 e das que foram feitas sob convênio:

Q:
```js
  db.consultas.aggregate([
  {
    $match: {
      "data_e_hora": {
        $gte: ISODate("2020-01-01T00:00:00Z"),
        $lte: ISODate("2020-12-31T23:59:59Z")
      }
    }
  },
  {
    $group: {
      _id: null,
      consultas: { $push: "$$ROOT" },
      media_valor: { $avg: "$consulta_valor.valor" }
    }
  },
  {
    $project: {
      _id: 0,
      consultas: 1,
      media_valor: 1
    }
  }
])
```
R: existem duas consultas apenas em 2020 e as duas não foram feitas sob convenio, o valor medio das duas consultas é de 200


  2. Todos os dados das internações que tiveram data de alta maior que a data prevista para a alta:

Q:
```js
db.internacoes.find({
  $expr: {
    $gt: ["$data_efetiva_alta", "$data_prevista_alta"]
  }
})
```
R: No Banco de Dados não existem internções com data de alta maior que a data prevista de alta.

  3. Receituário completo da primeira consulta registrada com receituário associado:

Q:
```js
db.consultas.find(
  { "receita_do_medico": { $exists: true, $ne: null } }
).sort(
  { "data_e_hora": 1 }
).limit(1)
```
R: 
```js
{
  "_id": {
    "$oid": "68251da55c2870d7ba999478"
  },
  "data_e_hora": {
    "$date": "2019-08-15T15:00:00.000Z"
  },
  "status": "1",
  "CRM_medico": "556677-SP",
  "CPF_paciente": "111.222.333-44",
  "especialidade_procurada": "Clínica Geral",
  "consulta_valor": {
    "valor": 150,
    "convenio": true,
    "numero_convenio": "CONV33445"
  },
  "receita_do_medico": {
    "medicamento_receitado": [
      "Dipirona"
    ],
    "dosagem": [
      "500mg"
    ],
    "duracao": [
      "5 dias"
    ],
    "indicacao": [
      "Febre"
    ],
    "observacao": "Tomar a cada 8 horas"
  },
  "encaminhamento": {
    "encaminhamento": false,
    "area_encaminhada": ""
  },
  "retorno": {
    "retorno_necessario": false,
    "data_do_retorno": "",
    "descricao": ""
  },
  "descricao": "Consulta para febre persistente",
  "internacao": true
}
```

  4. Todos os dados da consulta de maior valor e também da de menor valor (ambas as consultas não foram realizadas sob convênio):

Q:

para mais cara:

```js
db.consultas.find(
  { "consulta_valor.convenio": false }
).sort(
  { "consulta_valor.valor": -1 }
).limit(1)
```

para mais barata:

```js
db.consultas.find(
  { "consulta_valor.convenio": false }
).sort(
  { "consulta_valor.valor": 1 }
).limit(1)
```

  5. Todos os dados das internações em seus respectivos quartos, calculando o total da internação a partir do valor de diária do quarto e o número de dias entre a entrada e a alta:

Q:
```js
db.internacoes.aggregate([
  {
    $lookup: {
      from: "quartos",
      localField: "numero_quarto",
      foreignField: "numero_quarto",
      as: "quarto_info"
    }
  },
  {
    $unwind: "$quarto_info"
  },
  {
    $addFields: {
      dataEntradaDate: { $dateFromString: { dateString: "$data_de_entrada" } },
      dataAltaDate: { $dateFromString: { dateString: "$data_efetiva_alta" } },
      valorDiarioNum: { $toDouble: "$quarto_info.valor_diario" }
    }
  },
  {
    $addFields: {
      diffDias: {
        $ceil: {
          $divide: [
            { $subtract: ["$dataAltaDate", "$dataEntradaDate"] },
            1000 * 60 * 60 * 24
          ]
        }
      }
    }
  },
  {
    $addFields: {
      totalInternacao: { $multiply: ["$diffDias", "$valorDiarioNum"] }
    }
  },
  {
    $project: {
      _id: 1,
      paciente_internado: 1,
      CPF_paciente: 1,
      COREN_do_infermeiro: 1,
      data_de_entrada: 1,
      data_prevista_alta: 1,
      data_efetiva_alta: 1,
      numero_quarto: 1,
      descricao: 1,
      quarto_info: 1,
      diffDias: 1,
      totalInternacao: 1
    }
  }
])
```
R: exemplo: 
```js
{
  _id: ObjectId('6821d83c7ce4386018e1d04d'),
  paciente_internado: true,
  CPF_paciente: '321.654.987-00',
  COREN_do_infermeiro: '333333-SP',
  data_de_entrada: '2018-01-05',
  numero_quarto: '201',
  data_prevista_alta: '2018-01-10',
  data_efetiva_alta: '2018-01-10',
  descricao: 'Internação para tratamento de asma.',
  quarto_info: {
    _id: ObjectId('6821d84d7ce4386018e1d055'),
    numero_quarto: '201',
    tipo_quarto: 'Enfermaria',
    valor_diario: '150.00',
    tipo: 'Coletivo',
    COREN_infermeiro_responsalvel: '444444-SP',
    COREN_infermeiro_responsalvel_2: '555555-SP',
    internacao: false
  },
  diffDias: 5,
  totalInternacao: 750
```

  6.Data, procedimento e número de quarto de internações em quartos do tipo “apartamento”:

Q:
```js
db.internacao.aggregate([
  {
    $lookup: {
      from: "quartos",
      localField: "numero_quarto",
      foreignField: "numero_quarto",
      as: "quarto_info"
    }
  },
  { 
    $unwind: "$quarto_info" 
  },
  { 
    $match: { "quarto_info.tipo_quarto": "Apartamento" } 
  },
  {
    $project: {
      _id: 0,
      data_de_entrada: 1,
      descricao: 1,
      numero_quarto: 1
    }
  }
])
```
R:
```
{
  data_de_entrada: '2016-03-15',
  numero_quarto: '101',
  descricao: 'Internação para tratamento de hipertensão.'
},

{
  data_de_entrada: '2019-05-20',
  numero_quarto: '101',
  descricao: 'Internação para controle de pressão arterial.'
}
```
  
  7. Nome do paciente, data da consulta e especialidade de todas as consultas em que os pacientes eram menores de 18 anos na data da consulta e cuja especialidade não seja “pediatria”, ordenando por data de realização da consulta:

Q:
```js
db.consultas.aggregate([
  {
    $lookup: {
      from: "pacientes",
      localField: "CPF_paciente",
      foreignField: "documentos.CPF",
      as: "paciente_info"
    }
  },
  { 
    $unwind: "$paciente_info" 
  },
  {
    $addFields: {
      dataNascimento: { $dateFromString: { dateString: "$paciente_info.data_de_nascimento" } },
      dataConsulta: "$data_e_hora"
    }
  },
  {
    $addFields: {
      idade: {
        $dateDiff: {
          startDate: "$dataNascimento",
          endDate: "$dataConsulta",
          unit: "year"
        }
      }
    }
  },
  {
    $match: {
      idade: { $lt: 18 },
      especialidade_procurada: { $ne: "Pediatria" }
    }
  },
  {
    $project: {
      _id: 0,
      nome_paciente: "$paciente_info.nome_paciente",
      data_consulta: "$data_e_hora",
      especialidade: "$especialidade_procurada"
    }
  },
  {
    $sort: { data_consulta: 1 }
  }
])
```
R: Não exite paciente menor de 18 anos no Banco :/

  8. Os nomes dos médicos, seus CRMs e a quantidade de consultas que cada um realizou:

```js
db.consultas.aggregate([
  {
    $group: {
      _id: "$CRM_medico",
      total_consultas: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "medicos",
      localField: "_id",
      foreignField: "documentos.CRM",
      as: "dados_medico"
    }
  },
  { $unwind: "$dados_medico" },
  {
    $project: {
      _id: 0,
      nome_medico: "$dados_medico.nome_medico",
      CRM: "$_id",
      total_consultas: 1
    }
  }
])
```
R: Todas as respostas trazem que cada medico teve 2 numero de consultas

  9. Todos os médicos que tenham "Gabriel" no nome:

```js
db.medicos.find({ nome_medico: /Gabriel/i })
```
R: não tem Gabriel :/

  10. Os nomes, CORENs e número de internações de enfermeiros que participaram de mais de uma internação:

Q:
```js
db.internacao.aggregate([
  {
    $group: {
      _id: "$COREN_do_infermeiro",
      total_internacoes: { $sum: 1 }
    }
  },
  {
    $lookup: {
      from: "enfermeiros",
      localField: "_id",
      foreignField: "documentos.COREN",
      as: "enfermeiro"
    }
  },
  { $unwind: "$enfermeiro" },
  {
    $project: {
      _id: 0,
      nome: "$enfermeiro.nome",
      COREN: "$_id",
      total_internacoes: 1
    }
  }
])
```
R: Como cada infermeiro só tem uma internação, fiz uma query apenas para trazer qual o total de internação de cada infermeiro. Cada um tem uma internação.
