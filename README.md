# 🏦 Finanças Config Resources

Repositório centralizado de configurações para o projeto de finanças. Este repositório contém todas as configurações dos microserviços que são consumidas pelo Spring Cloud Config Server.

## 📋 Sobre o Repositório

O **Finanças Config Resources** é um repositório Git que serve como fonte de configuração para o Spring Cloud Config Server. Ele contém:

- ✅ **Configurações por ambiente** (dev, prod, hml)
- ✅ **Configurações por aplicação** (extrato-api, etc.)
- ✅ **Configurações centralizadas** e versionadas
- ✅ **Histórico de mudanças** via Git
- ✅ **Rollback fácil** de configurações

## 🏗️ Estrutura do Repositório

```
financas-config-resources/
├── README.md
├── extrato-api/
│   ├── extrato-api.yml          # Configuração base
│   ├── extrato-api-dev.yml      # Desenvolvimento
│   ├── extrato-api-prod.yml     # Produção
│   └── extrato-api-hml.yml      # Homologação
├── financas-frontend/
│   ├── financas-frontend.yml
│   ├── financas-frontend-dev.yml
│   └── financas-frontend-prod.yml
└── shared/
    ├── database.yml              # Configurações de banco
    ├── rabbitmq.yml              # Configurações do RabbitMQ
    └── observability.yml         # Configurações de monitoramento
```

## 🚀 Como Usar

### 1. **Adicionar Nova Configuração**

Para adicionar uma nova configuração:

1. **Crie o arquivo** na pasta da aplicação:
```bash
# Exemplo: observability.yml
touch extrato-api/observability.yml
```

2. **Adicione as configurações:**
```yaml
# observability.yml
spring:
  profiles:
    active: prod
  datasource:
    url: jdbc:mysql://prod-db:3306/financas_db
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

3. **Commit e push:**
```bash
git add .
git commit -m "feat: add production config for extrato-api"
git push origin main
```

### 2. **Atualizar Configuração Existente**

Para atualizar uma configuração:

1. **Edite o arquivo** desejado
2. **Commit a mudança:**
```bash
git add extrato-api/banks-conf.yml
git commit -m "fix: update database connection timeout"
git push origin main
```

3. **Refresh no Config Server** (se necessário):
```bash
curl -X POST http://localhost:8889/actuator/refresh
```

## 📚 Configurações Disponíveis

### **Extrato API**

#### **Configuração Base** (`extrato-api.yml`)
- Configurações comuns a todos os ambientes
- Configurações de parser (BB e Itaú)
- Configurações de observabilidade

#### **Desenvolvimento** (`extrato-api-dev.yml`)
- Logging em DEBUG
- DDL auto-update
- Configurações de desenvolvimento

#### **Produção** (`extrato-api-prod.yml`)
- Logging otimizado
- DDL validate
- Configurações de performance

#### **Homologação** (`extrato-api-hml.yml`)
- Configurações intermediárias
- Testes de integração
- Validações rigorosas

### **Configurações Compartilhadas**

#### **Database** (`shared/database.yml`)
- Configurações comuns de banco
- Pool de conexões
- Configurações de transação

#### **RabbitMQ** (`shared/rabbitmq.yml`)
- Configurações de filas
- Configurações de retry
- Configurações de dead letter

#### **Observability** (`shared/observability.yml`)
- Configurações do Prometheus
- Configurações do Zipkin
- Configurações de logging

## 🔧 Convenções

### **Nomenclatura de Arquivos**
- `{aplicacao}.yml` - Configuração base
- `{aplicacao}-{ambiente}.yml` - Configuração específica do ambiente
- `shared/{recurso}.yml` - Configurações compartilhadas

### **Estrutura YAML**
```yaml
# Sempre incluir o profile ativo
spring:
  profiles:
    active: {ambiente}

# Agrupar configurações por categoria
spring:
  datasource: # Banco de dados
  rabbitmq:   # Mensageria
  jpa:        # JPA/Hibernate

# Configurações customizadas
parser:       # Parsers de extrato
bancos:       # Bancos suportados
```

### **Variáveis de Ambiente**
- Use `${VARIAVEL:valor_padrao}` para variáveis
- Documente todas as variáveis necessárias
- Use nomes descritivos e consistentes

## 📊 Versionamento

### **Tags de Versão**
```bash
# Criar tag para release
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0

# Listar tags
git tag -l
```

### **Branches**
- `main` - Branch principal (produção)
- `develop` - Branch de desenvolvimento
- `feature/*` - Branches de feature
- `hotfix/*` - Branches de correção

## 🔐 Segurança

### **Secrets**
- **NUNCA** commite senhas ou tokens
- Use variáveis de ambiente para secrets
- Use `${SECRET_VAR}` nos arquivos de configuração

### **Exemplo de Configuração Segura:**
```yaml
spring:
  datasource:
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  rabbitmq:
    password: ${RABBITMQ_PASSWORD}
```

## 🚀 Integração com Config Server

### **URLs de Acesso**
- **Config Server:** http://localhost:8889
- **Configuração:** `/{aplicacao}/{profile}/{label}`
- **Exemplo:** http://localhost:8889/extrato-api/dev/main

### **Exemplo de Consumo**
```bash
# Buscar configuração do extrato-api em dev
curl http://localhost:8889/extrato-api/dev/main

# Buscar configuração específica
curl http://localhost:8889/extrato-api/prod/main
```

## 📝 Contribuição

### **Processo de Contribuição**
1. **Fork** o repositório
2. **Crie uma branch** para sua feature
3. **Faça as mudanças** necessárias
4. **Teste** as configurações
5. **Commit** com mensagem descritiva
6. **Push** para sua branch
7. **Abra um Pull Request**

### **Padrões de Commit**
```bash
# Adicionar nova configuração
git commit -m "feat: add production config for extrato-api"

# Atualizar configuração existente
git commit -m "fix: update database connection pool size"

# Correção de bug
git commit -m "fix: correct RabbitMQ retry configuration"

# Documentação
git commit -m "docs: update README with new configuration examples"
```

## 🛠️ Ferramentas Recomendadas

### **Edição**
- **VS Code** com extensão YAML
- **IntelliJ IDEA** com suporte a Spring
- **Vim/Emacs** com syntax highlighting

### **Validação**
- **YAML Lint** para validar sintaxe
- **Spring Boot Config** para validar propriedades
- **Git Hooks** para validação automática

## 📞 Suporte

Para dúvidas ou problemas:

1. **Verifique** a documentação
2. **Consulte** os exemplos existentes
3. **Abra uma issue** no repositório
4. **Entre em contato** com a equipe

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo `LICENSE` para mais detalhes.

---

**🎯 Este repositório é essencial para o funcionamento do projeto de finanças. Mantenha-o sempre atualizado e bem documentado!**
