# 📚 Como Usar as Configurações

Este documento explica como usar as configurações do repositório `financas-config-resources`.

## 🚀 Configuração Inicial

### 1. **Configurar o Config Server**

O Config Server deve estar configurado para apontar para este repositório:

```properties
# application.properties do Config Server
spring.cloud.config.server.git.uri=https://github.com/JeanCSDeSouza/financas-config-resources
spring.cloud.config.server.git.clone-on-start=true
spring.cloud.config.server.git.default-label=main
spring.cloud.config.server.git.search-paths=configs
```

### 2. **Configurar o Cliente (Extrato API)**

No seu microserviço, adicione as dependências:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

Crie o arquivo `bootstrap.yml`:

```yaml
spring:
  application:
    name: extrato-api
  cloud:
    config:
      uri: http://financas-config:8889
      fail-fast: true
      retry:
        initial-interval: 1000
        max-attempts: 6
        max-interval: 2000
        multiplier: 1.1
```

## 🔧 Uso das Configurações

### **1. Configuração por Ambiente**

#### **Desenvolvimento:**
```bash
# Variáveis de ambiente
export SPRING_PROFILES_ACTIVE=dev
export DB_HOST=localhost
export DB_PORT=3306
export DB_NAME=financas_db
export DB_USERNAME=appuser
export DB_PASSWORD=apppassword
```

#### **Produção:**
```bash
# Variáveis de ambiente
export SPRING_PROFILES_ACTIVE=prod
export DB_HOST=financas-db.cluster-xyz.us-east-1.rds.amazonaws.com
export DB_PORT=3306
export DB_NAME=financas_db
export DB_USERNAME=${DB_USERNAME}
export DB_PASSWORD=${DB_PASSWORD}
```

### **2. Acessando Configurações**

#### **Via Config Server:**
```bash
# Buscar configuração completa
curl http://localhost:8889/extrato-api/dev/main

# Buscar configuração específica
curl http://localhost:8889/extrato-api/prod/main
```

#### **Via Aplicação:**
```java
@Value("${spring.datasource.url}")
private String databaseUrl;

@Value("${parser.config.banco-do-brasil.name}")
private String bancoDoBrasilName;
```

### **3. Configurações Compartilhadas**

As configurações em `shared/` são aplicadas automaticamente:

- `shared/database.yml` - Configurações de banco
- `shared/rabbitmq.yml` - Configurações do RabbitMQ
- `shared/observability.yml` - Configurações de monitoramento

## 📝 Adicionando Novas Configurações

### **1. Para uma Aplicação Existente**

```bash
# Criar arquivo de configuração
touch extrato-api/extrato-api-staging.yml

# Adicionar configurações
cat > extrato-api/extrato-api-staging.yml << EOF
spring:
  profiles:
    active: staging
  datasource:
    url: jdbc:mysql://staging-db:3306/financas_db
    username: \${DB_USERNAME}
    password: \${DB_PASSWORD}
EOF

# Commit e push
git add .
git commit -m "feat: add staging configuration for extrato-api"
git push origin main
```

### **2. Para uma Nova Aplicação**

```bash
# Criar pasta da aplicação
mkdir financas-notifications

# Criar arquivos de configuração
touch financas-notifications/financas-notifications.yml
touch financas-notifications/financas-notifications-dev.yml
touch financas-notifications/financas-notifications-prod.yml

# Adicionar configurações básicas
cat > financas-notifications/financas-notifications.yml << EOF
spring:
  application:
    name: financas-notifications

# Configurações específicas da aplicação
notifications:
  email:
    enabled: true
    smtp:
      host: \${SMTP_HOST:localhost}
      port: \${SMTP_PORT:587}
      username: \${SMTP_USERNAME}
      password: \${SMTP_PASSWORD}
EOF
```

## 🔄 Atualizando Configurações

### **1. Atualização Simples**

```bash
# Editar arquivo
vim extrato-api/extrato-api-dev.yml

# Commit
git add extrato-api/extrato-api-dev.yml
git commit -m "fix: update database connection timeout"
git push origin main
```

### **2. Atualização com Rollback**

```bash
# Fazer mudança
git add .
git commit -m "feat: add new configuration"
git push origin main

# Se algo der errado, fazer rollback
git log --oneline
git revert <commit-hash>
git push origin main
```

### **3. Refresh do Config Server**

```bash
# Refresh manual (se necessário)
curl -X POST http://localhost:8889/actuator/refresh

# Refresh de aplicação específica
curl -X POST http://localhost:8080/actuator/refresh
```

## 🧪 Testando Configurações

### **1. Validação Local**

```bash
# Validar sintaxe YAML
yamllint extrato-api/extrato-api-dev.yml

# Testar configuração
curl http://localhost:8889/extrato-api/dev/main | jq
```

### **2. Teste de Integração**

```bash
# Iniciar Config Server
cd financas-configuration
mvn spring-boot:run

# Testar configuração
curl http://localhost:8889/extrato-api/dev/main
```

## 🔐 Segurança

### **1. Secrets**

**NUNCA** commite senhas ou tokens:

```yaml
# ❌ ERRADO
spring:
  datasource:
    password: minha-senha-secreta

# ✅ CORRETO
spring:
  datasource:
    password: ${DB_PASSWORD}
```

### **2. Variáveis de Ambiente**

Configure as variáveis de ambiente:

```bash
# Desenvolvimento
export DB_PASSWORD=dev-password
export RABBITMQ_PASSWORD=dev-rabbitmq-password

# Produção (use um gerenciador de secrets)
export DB_PASSWORD=$(aws secretsmanager get-secret-value --secret-id prod/db/password --query SecretString --output text)
```

## 📊 Monitoramento

### **1. Health Check**

```bash
# Verificar saúde do Config Server
curl http://localhost:8889/actuator/health

# Verificar configurações carregadas
curl http://localhost:8889/actuator/configprops
```

### **2. Logs**

```bash
# Ver logs do Config Server
docker logs financas-config

# Ver logs da aplicação
docker logs extrato-api
```

## 🚨 Troubleshooting

### **1. Configuração não carregada**

```bash
# Verificar se o Config Server está rodando
curl http://localhost:8889/actuator/health

# Verificar se a aplicação está configurada corretamente
curl http://localhost:8080/actuator/env | jq '.propertySources'
```

### **2. Erro de conexão**

```bash
# Verificar conectividade
ping financas-config
telnet financas-config 8889

# Verificar DNS
nslookup financas-config
```

### **3. Configuração incorreta**

```bash
# Verificar sintaxe YAML
yamllint extrato-api/extrato-api-dev.yml

# Verificar propriedades Spring
curl http://localhost:8889/extrato-api/dev/main | jq '.propertySources[].source'
```

## 📞 Suporte

Para dúvidas ou problemas:

1. **Verifique** este documento
2. **Consulte** os exemplos existentes
3. **Abra uma issue** no repositório
4. **Entre em contato** com a equipe

---

**🎯 Lembre-se: As configurações são a base do funcionamento dos microserviços. Mantenha-as sempre atualizadas e bem documentadas!**
