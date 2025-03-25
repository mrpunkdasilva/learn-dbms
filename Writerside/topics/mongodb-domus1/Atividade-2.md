# 2. CREATE - Criar Banco, Coleção e Documentos  

## **Criar um Banco de Dados**

```javascript
use lab_crud
```

![Exemplo de criação de um banco de dados chamado "lab_crud"](1742909967039.png)





<note>

O MongoDB cria o banco quando o primeiro documento é inserido.

</note>

## **Criar uma Coleção**  

```javascript
db.createCollection("clientes")
```


![Exemplo de criação de uma coleção chamada "clientes"](1742909993962.png)



<note>

Se não existir, o MongoDB cria automaticamente ao inserir dados.

</note>


## Inserir Documentos (INSERT)

```javascript
// Inserir um documento
db.clientes.insertOne({
  nome: "João Silva",
  idade: 30,
  email: "joao@email.com",
  cidade: "São Paulo"
})

// Inserir múltiplos documentos
db.clientes.insertMany([
  { nome: "Maria Souza", idade: 25, email: "maria@email.com", cidade: "Rio de Janeiro" },
  { nome: "Carlos Oliveira", idade: 35, email: "carlos@email.com", cidade: "Belo Horizonte" }
])
```

![Exemplos de inserção de documentos na coleção "clientes"](1742910045471.png)
