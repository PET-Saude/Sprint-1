# Guia de Versionamento e Fluxo de Trabalho (GitFlow)

Este documento detalha o processo de controle de versionamento utilizado em nosso projeto, seguindo o padrão **GitFlow**.  
O objetivo é garantir que o desenvolvimento, as correções de bugs e as novas funcionalidades sejam organizados, rastreáveis e seguros.



## 1. Convenções de Nomenclatura de Branches

Para manter a organização, seguimos uma convenção rigorosa para os nomes dos branches.  
Use sempre nomes descritivos, em minúsculas e separados por hífens.

- **main**: Branch principal e de produção. O código aqui deve sempre estar estável e pronto para implantação.  
- **dev**: Branch de desenvolvimento. É o branch de integração principal, onde todas as novas funcionalidades são mescladas antes de irem para `main`.  
- **feature/<nome-da-feature>**: Para o desenvolvimento de novas funcionalidades.  
  - Exemplo: `feature/adicionar-autenticacao`  
- **bugfix/<nome-do-bug>**: Para correções de bugs.  
  - Exemplo: `bugfix/ajustar-layout-pagina-inicial`  
- **hotfix/<nome-do-bug>**: Para correções emergenciais em produção. Criado a partir de `main`.  
  - Exemplo: `hotfix/corrigir-erro-login`  


## 2. Fluxo de Trabalho (Workflow)

Este é o fluxo padrão para o desenvolvimento de uma nova funcionalidade ou correção de bug.

### 2.1. Iniciar o Trabalho

Sempre atualize seu branch local `main` e `dev` antes de começar:

```bash
git checkout main
git pull origin main
git checkout dev
git pull origin dev
```

Crie um novo branch a partir de dev (ou main para hotfixes):

```bash
# Para uma nova feature
git checkout -b feature/minha-nova-feature dev

# Para uma correção de bug
git checkout -b bugfix/corrigir-validacao dev
```

### 2.2. Desenvolver e Fazer Commits

Trabalhe no seu branch e faça commits com frequência.

As mensagens de commit devem ser claras e descritivas, usando o imperativo:

- **Bom exemplo**: feat: Adiciona botão de login na página inicial
- **Mau exemplo**: Mudanças no login

Empurre (push) seu branch para o repositório remoto:
```bash
git push origin <seu-branch>
```

### 2.3. Criar um Pull Request (PR)

Ao finalizar o trabalho:

**1.** Crie um Pull Request no GitHub (ou outra plataforma) para mesclar seu branch para dev.

**2.** Preencha o template de PR, descrevendo as mudanças, o problema resolvido e observações relevantes.

**3.** Solicite revisão de código. O PR deve ser aprovado por pelo menos um colega antes de ser mesclado.

### 2.4. Resolução de Conflitos

Caso seu PR tenha conflitos:

**1.** Não resolva diretamente na interface do GitHub.

**2.** Atualize seu branch local com as últimas mudanças do dev:
```bash
git checkout <seu-branch>
git pull origin dev
```

**3.** Resolva os conflitos no editor, faça um novo commit e empurre as mudanças:
```bash
git add .
git commit -m "Resolve conflitos com dev"
git push origin <seu-branch>
```

### 2.5. Finalizar o Pull Request

**1.** Após aprovação e resolução de conflitos, o revisor pode mesclar (merge) o PR.

**2.** Após o merge, o branch feature/ ou bugfix/ pode ser deletado:
```bash
git branch -d feature/minha-nova-feature
git push origin --delete feature/minha-nova-feature
```

### 3. Boas Práticas

- Commits pequenos e atômicos: Cada commit deve ser uma mudança lógica e única.
- Mantenha seus branches atualizados: Sincronize seu branch de trabalho com dev frequentemente para evitar grandes conflitos.
- Não mescle sem revisão: A revisão de código é crucial para a qualidade do projeto.
- Não trabalhe diretamente em dev ou main: Apenas commits de merge são permitidos nesses branches.
- Seja consistente: Siga as convenções estabelecidas para garantir clareza e rastreabilidade do histórico do projeto.
