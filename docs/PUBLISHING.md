# Publishing Guide - Aegis Test Interfaces

## Overview

Este guia descreve o processo completo para publicar as bibliotecas **Java** e **Python** no GitHub Packages.

As bibliotecas são publicadas automaticamente via GitHub Actions quando você cria uma **Release**.

---

## Versioning

Ambas as bibliotecas compartilham a mesma versão para manter sincronização:

- **Java**: `wrappers/java/pom.xml` → `<version>X.Y.Z</version>`
- **Python**: `wrappers/python/pyproject.toml` → `version = "X.Y.Z"`

**Versão atual**: `0.0.1`

---

## Pre-Release Checklist

Antes de criar uma release, verifique:

### 1. Versões Sincronizadas

```bash
# Verificar Java
cat wrappers/java/pom.xml | grep "<version>"
# Deve retornar: <version>0.0.1</version>

# Verificar Python
cat wrappers/python/pyproject.toml | grep "^version"
# Deve retornar: version = "0.0.1"
```

Se as versões forem diferentes, sincronize-as antes de continuar.

### 2. Commit Latest Changes

```bash
git add .
git commit -m "Prepare release v0.0.1"
git push origin main
```

### 3. Validação

Certifique-se de que a branch `main` passou em todos os checks:
- ✅ Validate workflow passou
- ✅ Não há conflitos
- ✅ Documentação atualizada

---

## Publishing Process

### Step 1: Create Release Tag

**Via Git CLI:**

```bash
git tag -a v0.0.1 -m "Release version 0.0.1

- Java wrapper for Aegis Test messaging interfaces
- Python wrapper for Aegis Test messaging interfaces
- Event schemas and topic definitions
- AsyncAPI 3.1.0 specification

Featuring:
- Type-safe Pub/Sub destinations
- Central registry pattern
- Full support for Java 21 and Python 3.13"

git push origin v0.0.1
```

**Via GitHub UI:**

1. Ir para [Releases](https://github.com/peguidotte/aegis-test-pubsub-interfaces/releases)
2. Clicar "Create a new release"
3. Preencher:
   - **Tag version**: `v0.0.1`
   - **Release title**: `v0.0.1`
   - **Description**: 
     ```markdown
     ## Release v0.0.1

     Initial release of Aegis Test messaging interfaces.

     ### Features
     - Java wrapper (Java 21+) with type-safe destinations
     - Python wrapper (Python 3.10+) with type hints
     - Event schemas (JSON Schema)
     - Topic definitions (YAML)
     - AsyncAPI 3.1.0 specification
     - GitHub Actions CI/CD pipelines

     ### What's Inside
     - `Destination` interface for messaging destinations
     - `Topics` registry with all available destinations
     - `SpecificationCreated` and `SpecificationRequested` events
     - Automatic wrapper generation from schemas

     ### Installation

     **Java (Maven):**
     ```xml
     <repository>
         <id>github</id>
         <url>https://maven.pkg.github.com/peguidotte/aegis-test-pubsub-interfaces</url>
     </repository>

     <dependency>
         <groupId>com.aegis.test</groupId>
         <artifactId>aegis-test-pubsub-interfaces</artifactId>
         <version>0.0.1</version>
     </dependency>
     ```

     **Python (pip from Git):**
     ```bash
     pip install git+https://github.com/peguidotte/aegis-test-pubsub-interfaces.git@v0.0.1#subdirectory=wrappers/python
     ```
     ```
   - **Is a pre-release**: ✅ Marcar (pois é v0.0.1)
4. Clicar "Publish release"

---

### Step 2: GitHub Actions Triggers

Quando você publica a release, o workflow **publish.yml** é acionado automaticamente:

```
Release Created
    ↓
publish.yml workflow triggers
    ↓
┌─────────────────┬─────────────────┐
│  Publish Java   │  Publish Python │
│   to Maven      │   to PyPI       │
└─────────────────┴─────────────────┘
    ↓
Upload to GitHub Packages
    ↓
Create Release Summary
```

Você pode acompanhar em:
- **Actions tab**: [GitHub Actions](https://github.com/peguidotte/aegis-test-pubsub-interfaces/actions)
- **Publish workflow**: Verifique o logs do workflow_run

---

### Step 3: Verification

Após a publicação, verifique se tudo funcionou:

#### Java Package

```bash
# Listar releases Maven
curl -s -H "Authorization: token YOUR_GITHUB_TOKEN" \
  https://maven.pkg.github.com/peguidotte/aegis-test-pubsub-interfaces/com/aegis/test/aegis-test-pubsub-interfaces/ | \
  grep -o 'aegis-test-pubsub-interfaces-[0-9.]*\.jar' | sort -u

# Ou verificar no GitHub:
# https://github.com/peguidotte/aegis-test-pubsub-interfaces/packages
```

#### Python Package

```bash
# Verificar no GitHub:
# https://github.com/peguidotte/aegis-test-pubsub-interfaces/packages
# (Na tab "Packages")
```

---

## Using the Published Libraries

### Java

#### 1. Configure Repository

Em seu `pom.xml`:

```xml
<repositories>
    <repository>
        <id>github</id>
        <name>GitHub Packages</name>
        <url>https://maven.pkg.github.com/peguidotte/aegis-test-pubsub-interfaces</url>
        <releases>
            <enabled>true</enabled>
        </releases>
    </repository>
</repositories>
```

#### 2. Add Dependency

```xml
<dependency>
    <groupId>com.aegis.test</groupId>
    <artifactId>aegis-test-pubsub-interfaces</artifactId>
    <version>0.0.1</version>
</dependency>
```

#### 3. Configure Authentication

Em `~/.m2/settings.xml`:

```xml
<servers>
    <server>
        <id>github</id>
        <username>YOUR_GITHUB_USERNAME</username>
        <password>YOUR_GITHUB_PERSONAL_TOKEN</password>
    </server>
</servers>
```

[Como gerar Personal Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

#### 4. Use in Code

```java
import com.aegis.test.interfaces.messaging.Destination;
import com.aegis.test.interfaces.messaging.Topics;

public class MyService {
    public void publish() {
        Destination dest = Topics.SPECIFICATION_CREATED;
        String topic = dest.getTopic();
        String subscription = dest.getSubscription("analytics");
        // Use in your Pub/Sub code...
    }
}
```

---

### Python

#### 1. Install from Git

```bash
pip install git+https://github.com/peguidotte/aegis-test-pubsub-interfaces.git@v0.0.1#subdirectory=wrappers/python
```

#### 2. Use in Code

```python
from aegis_interfaces.messaging import Topics

class MyService:
    def publish(self):
        dest = Topics.SPECIFICATION_CREATED
        topic = dest.topic
        subscription = dest.get_subscription("analytics")
        # Use in your Pub/Sub code...
```

---

## Troubleshooting

### Maven: 401 Unauthorized

```
[ERROR] Failed to fetch data from maven.pkg.github.com
[ERROR] 401 Unauthorized
```

**Solução**: Verifique credenciais em `~/.m2/settings.xml`

### Maven: Cannot access Repository

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-deploy-plugin
```

**Solução**: Certifique-se de que o `pom.xml` tem `<distributionManagement>` configurado:

```xml
<distributionManagement>
    <repository>
        <id>github</id>
        <name>GitHub Packages</name>
        <url>https://maven.pkg.github.com/peguidotte/aegis-test-pubsub-interfaces</url>
    </repository>
</distributionManagement>
```

### Python: Permission Denied

```
ERROR: HTTP Error 403: Forbidden
```

**Solução**: Verifique token do GitHub com permissões `read:packages` e `write:packages`

---

## Next Release (v0.0.2+)

Para próximas releases, siga o mesmo processo:

1. **Update versions**:
   ```bash
   # Update pom.xml
   mvn versions:set -DnewVersion=0.0.2

   # Update pyproject.toml
   # (manual edit no momento)
   ```

2. **Commit e push**:
   ```bash
   git add wrappers/
   git commit -m "Bump version to 0.0.2"
   git push origin main
   ```

3. **Create release**:
   ```bash
   git tag v0.0.2
   git push origin v0.0.2
   ```

---

## References

- [GitHub Packages Maven Registry](https://docs.github.com/en/packages/working-with-a-maven-registry)
- [GitHub Packages Python Registry](https://docs.github.com/en/packages/working-with-a-python-registry)
- [GitHub Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
- [Semantic Versioning](https://semver.org/)

---

**Last Updated**: February 14, 2026  
**Version**: v0.0.1
