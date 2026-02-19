

# 🧠 Social Network Graph — Neo4j + APOC

Este projeto implementa uma **simulação completa de rede social** utilizando **banco de dados em grafo (Neo4j)**, gerando **+1000 usuários fictícios**, posts, amizades, curtidas, comentários, compartilhamentos, grupos e **recomendações automáticas de amizade**, criando uma **estrutura altamente realista e escalável**.

O objetivo é estudar **modelagem de grafos sociais**, **análise de redes complexas**, **descoberta de padrões**, **recomendação social** e **visualização gráfica de conexões**.

---

# 🚀 Tecnologias

* Neo4j
* Cypher Query Language (CQL)
* APOC Procedures
* Neo4j Browser / Bloom

---

# 🏗 Arquitetura do Grafo

## Nós (Nodes)

| Label   | Descrição        |
| ------- | ---------------- |
| `User`  | Usuários da rede |
| `Post`  | Publicações      |
| `Group` | Grupos sociais   |

---

## Relacionamentos

| Tipo            | Descrição           |
| --------------- | ------------------- |
| `:FRIENDS_WITH` | Amizade             |
| `:CREATED`      | Criou post          |
| `:LIKED`        | Curtiu              |
| `:COMMENTED`    | Comentou            |
| `:SHARED`       | Compartilhou        |
| `:MEMBER_OF`    | Membro do grupo     |
| `:RECOMMENDED`  | Sugestão de amizade |

---

# 📌 Pré-requisitos

APOC instalado:

```cypher
RETURN apoc.version();
```

---

# 🧱 Constraints — Integridade + Performance

```cypher
CREATE CONSTRAINT user_cpf_unique IF NOT EXISTS
FOR (u:User) REQUIRE u.cpf IS UNIQUE;

CREATE CONSTRAINT post_id_unique IF NOT EXISTS
FOR (p:Post) REQUIRE p.id IS UNIQUE;

CREATE CONSTRAINT group_id_unique IF NOT EXISTS
FOR (g:Group) REQUIRE g.id IS UNIQUE;
```

---

# 👥 Criação de +1000 Usuários (CPF Fictício Único)

```cypher
WITH [
 'Rian','Giulia','Lucas','Ana','Pedro','Mariana','Bruno','Carla','Felipe',
 'Julia','Rafael','Bianca','João','Camila','Daniel','Larissa','Gustavo',
 'Beatriz','Matheus','Isabela','Caio','Leticia','Henrique','Fernanda',
 'Vinicius','Natalia','Diego','Paula','Thiago','Aline','Igor','Luana'
] AS nomes

UNWIND range(1,1000) AS id
WITH id, nomes, apoc.text.random(3,'0123456789') AS cpf1,
     apoc.text.random(3,'0123456789') AS cpf2,
     apoc.text.random(3,'0123456789') AS cpf3,
     apoc.text.random(2,'0123456789') AS cpf4

CREATE (:User {
    id: id,
    nome: nomes[id % size(nomes)],
    sobrenome: apoc.text.capitalize(apoc.text.random(6,'abcdefghijklmnopqrstuvwxyz')),
    idade: 18 + (id % 45),
    email: toLower(nomes[id % size(nomes)]) + id + '@email.com',
    cpf: cpf1 + '.' + cpf2 + '.' + cpf3 + '-' + cpf4,
    criado_em: datetime()
});
```

---

# 🤝 Gerar Amizades (Rede Social Realista)

```cypher
MATCH (u:User)
WITH u
LIMIT 1000
CALL {
  WITH u
  MATCH (o:User)
  WHERE o <> u
  RETURN o
  ORDER BY rand()
  LIMIT 15
}
CREATE (u)-[:FRIENDS_WITH {since: date()}]->(o);
```

---

# 📝 Gerar Posts

```cypher
MATCH (u:User)
WITH u
UNWIND range(1, 5) AS i
CREATE (:Post {
  id: apoc.create.uuid(),
  conteudo: 'Post #' + i + ' de ' + u.nome,
  criado_em: datetime()
})<-[:CREATED]-(u);
```

---

# ❤️ Curtidas

```cypher
MATCH (u:User), (p:Post)
WITH u,p WHERE rand() < 0.12
CREATE (u)-[:LIKED {em: datetime()}]->(p);
```

---

# 💬 Comentários

```cypher
MATCH (u:User), (p:Post)
WITH u,p WHERE rand() < 0.06
CREATE (u)-[:COMMENTED {
  texto: 'Comentário de ' + u.nome,
  em: datetime()
}]->(p);
```

---

# 🔁 Compartilhamentos

```cypher
MATCH (u:User), (p:Post)
WITH u,p WHERE rand() < 0.04
CREATE (u)-[:SHARED {em: datetime()}]->(p);
```

---

# 👨‍👩‍👧‍👦 Criar Grupos

```cypher
UNWIND range(1,50) AS id
CREATE (:Group {
  id: id,
  nome: 'Grupo ' + id,
  criado_em: datetime()
});
```

---

# 👥 Usuários em Grupos

```cypher
MATCH (u:User), (g:Group)
WITH u,g WHERE rand() < 0.15
CREATE (u)-[:MEMBER_OF]->(g);
```

---

# 🤖 Sistema de Recomendação de Amizades

```cypher
MATCH (u:User)-[:FRIENDS_WITH]->(:User)-[:FRIENDS_WITH]->(fof:User)
WHERE NOT (u)-[:FRIENDS_WITH]->(fof) AND u <> fof
WITH u, fof, count(*) AS conexoes
WHERE conexoes >= 3
CREATE (u)-[:RECOMMENDED {peso: conexoes}]->(fof);
```

---

# 🔍 Consultas Avançadas

## Pessoas com conexões em comum

```cypher
MATCH (u1:User)-[r1]-(n)-[r2]-(u2:User)
RETURN u1, r1, n, r2, u2
LIMIT 50;
```

---

## Cadeia social complexa (6 pessoas conectadas)

```cypher
MATCH 
(a:User)-[:COMMENTED]->(p:Post)<-[:LIKED]-(b:User),
(b)-[:FRIENDS_WITH]-(c:User),
(c)-[:FRIENDS_WITH]-(d:User),
(d)-[:CREATED]->(p)
RETURN a,b,c,d,p
LIMIT 20;
```

---

## Usuários conectados via posts

```cypher
MATCH path=(u1:User)-[:LIKED|COMMENTED]->(p:Post)<-[:LIKED|COMMENTED]-(u2:User)
RETURN path
LIMIT 30;
```

---

# 📊 Visualização em Grafo

```cypher
MATCH path=(u:User)-[:LIKED|COMMENTED|CREATED|FRIENDS_WITH*1..3]-(n)
RETURN path
LIMIT 50;
```

---

# 📈 Métricas e Validação

## Total de usuários

```cypher
MATCH (u:User) RETURN count(u);
```

---

## Usuários mais conectados

```cypher
MATCH (u:User)-[:FRIENDS_WITH]->()
RETURN u.nome, count(*) AS total
ORDER BY total DESC
LIMIT 10;
```

---

## Melhores recomendações

```cypher
MATCH (u)-[r:RECOMMENDED]->(o)
RETURN u.nome, o.nome, r.peso
ORDER BY r.peso DESC
LIMIT 10;
```

---

# 🚀 Escalabilidade

Para aumentar a carga:

```cypher
range(1,1000)
```

➡ Troque para:

```cypher
range(1,10000)
```

Ou:

```cypher
range(1,50000)
```

Suporta **milhões de relações tranquilamente**.

---

# 🧠 Casos de Uso Reais

* Análise de influência social
* Sistemas de recomendação
* Detecção de comunidades
* Clustering social
* Grafos de amizade
* Caminhos mínimos
* Sugestão automática de conexões

---

# 👨‍💻 Autor

**Rian Gabriel Pires Barbalha**
Back-end Java | Graph Databases | IA | Engenharia de Prompts

📧 Email: [riangabrielpiresbarbalha@gmail.com](mailto:riangabrielpiresbarbalha@gmail.com)
🌐 GitHub: [https://github.com/rmythzl](https://github.com/rmythzl)

