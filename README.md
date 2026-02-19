🧠 Social Graph Database — Neo4j

Projeto de banco de dados em grafo usando Neo4j, simulando uma rede social realista, com 500 usuários, posts, grupos e múltiplos tipos de interações como amizades, curtidas e comentários.

O objetivo é estudar grafos, análise de conexões sociais, descoberta de padrões e visualização de relacionamentos complexos.

📌 Tecnologias Utilizadas

Neo4j

Cypher Query Language (CQL)

Neo4j Browser / Neo4j Bloom (visualização gráfica)

🏗 Estrutura do Grafo
Nós (Nodes)
Label	Descrição
User	Usuários da rede
Post	Publicações
Group	Grupos sociais
Relacionamentos
Tipo	Significado
:FRIEND	Amizade
:POSTED	Criou um post
:LIKED	Curtiu
:COMMENTED	Comentou
:MEMBER_OF	Membro de grupo
👥 Criação dos Usuários (500)
UNWIND range(1,500) AS id
CREATE (:User {
  id: id,
  name: 'User_' + id,
  age: 18 + (id % 40)
});

📝 Criação de Posts
MATCH (u:User)
WITH u, rand() AS r
WHERE r < 0.4
CREATE (u)-[:POSTED]->(:Post {
  id: apoc.create.uuid(),
  content: 'Post de ' + u.name,
  createdAt: datetime()
});

🔗 Criação de Relacionamentos
Amizades
MATCH (u1:User), (u2:User)
WHERE u1 <> u2 AND rand() < 0.02
MERGE (u1)-[:FRIEND]-(u2);

Curtidas
MATCH (u:User), (p:Post)
WHERE rand() < 0.05
MERGE (u)-[:LIKED]->(p);

Comentários
MATCH (u:User), (p:Post)
WHERE rand() < 0.03
MERGE (u)-[:COMMENTED]->(p);

Grupos
UNWIND range(1,20) AS id
CREATE (:Group {name:'Group_' + id});

MATCH (u:User), (g:Group)
WHERE rand() < 0.08
MERGE (u)-[:MEMBER_OF]->(g);

🔍 Consultas Principais (FUNCIONAIS)
🔹 Visualizar usuários em grafo
MATCH (u:User)
RETURN u
LIMIT 50;

🔹 Ver relações entre 4 usuários específicos
MATCH (u:User)
WHERE u.name IN ['Rian','Giulia','Alice','Bruno']
WITH collect(u) AS users

MATCH (common)
WHERE all(x IN users WHERE (x)--(common))
RETURN users, common;

🔹 Encontrar pessoas com relações em comum (até 5)
MATCH (u1:User)-[r1]-(n)-[r2]-(u2:User)
WHERE u1 <> u2
RETURN u1, r1, n, r2, u2
LIMIT 25;

📊 Visualização em Grafo (usuários + posts + conexões)
MATCH path=(u:User)-[:LIKED|COMMENTED|FRIEND|POSTED*1..3]-(n)
RETURN path
LIMIT 50;

🧠 Consulta Avançada — Cadeia Social Complexa

"6 pessoas conectadas: um comentou, outro curtiu, outro é amigo de quem criou o post."

MATCH 
(a:User)-[:COMMENTED]->(p:Post)<-[:LIKED]-(b:User),
(b)-[:FRIEND]-(c:User),
(c)-[:FRIEND]-(d:User),
(d)-[:POSTED]->(p)
RETURN a, b, c, d, p
LIMIT 10;

🔗 Usuários conectados através de interações em posts
MATCH path=(u1:User)-[:LIKED|COMMENTED]->(p:Post)<-[:LIKED|COMMENTED]-(u2:User)
RETURN path
LIMIT 20;

🔥 Descobrir comunidades naturais
CALL gds.louvain.stream({
  nodeProjection: 'User',
  relationshipProjection: {
    FRIEND: {type:'FRIEND', orientation:'UNDIRECTED'}
  }
})
YIELD nodeId, communityId
RETURN gds.util.asNode(nodeId).name AS user, communityId
ORDER BY communityId;

📈 Visualização Gráfica

No Neo4j Browser, use o modo Graph para visualizar:

Padrões sociais

Comunidades

Cadeias de influência

Conexões indiretas

🚀 Objetivo do Projeto

Este projeto foi criado para:

Aprender bancos de dados em grafo

Simular redes sociais reais

Executar consultas complexas

Explorar análise de relacionamentos

Gerar visualizações gráficas avançadas

📌 Possíveis Expansões

Sistema de recomendações

Detecção de influência social

Detecção de clusters

Ranking de usuários mais ativos

Caminhos mínimos entre pessoas

👨‍💻 Autor

Rian Gabriel Pires Barbalha
Desenvolvedor Back-end Java | Graph Databases | Engenharia de Prompts | Dados

📧 Email: riangabrielpiresbarbalha@gmail.com

🌐 GitHub: https://github.com/rmythzl
