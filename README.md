# Documentação: Fluxo de Trabalho Git Essencial e Hooks de Qualidade

Este guia prático ajuda você a dominar o fluxo de trabalho essencial do Git — **commits, branches e merges** — e a configurar **Git Hooks (pré-commit)** usando Python para garantir a qualidade e a segurança do seu código.

Neste exemplo, o hook irá **bloquear o commit** se você acidentalmente incluir informações sensíveis (como chaves de API, senhas ou tokens) ou palavras de validação diretamente nos arquivos do seu projeto. A prática correta é armazenar esses dados em arquivos de ambiente (`.env`) e ignorar o arquivo `.env` para que ele não seja enviado ao repositório.

## Pré-requisitos

  * **Git instalado:** Verifique se o Git está instalado em sua máquina com `git --version`.
  * **Editor de código:** Um editor como o VS Code.
  * **Python 3:** Certifique-se de que o Python 3 está instalado.

-----

## 1\. Configuração Inicial do Projeto

1.  Crie uma nova pasta e inicialize um repositório Git.

    ```bash
    mkdir projeto-git-pratica
    cd projeto-git-pratica
    git init
    ```

2.  Configure suas informações de usuário.

    ```bash
    git config --global user.name "Seu Nome"
    git config --global user.email "seu.email@exemplo.com"
    ```

-----

## 2\. Praticando Commits, Branches e Merges

Siga os passos abaixo para se familiarizar com o fluxo de trabalho básico do Git antes de configurar o hook.

### 2.1. Praticando Commits

1.  Crie um arquivo `README.md` e faça o primeiro commit.

    ```bash
    echo "# Projeto de Prática Git" >> README.md
    git add README.md
    git commit -m "feat: Adiciona README inicial do projeto"
    ```

### 2.2. Praticando Branches e Merges

1.  Crie e mude para uma nova branch chamada `feature/adicionar-funcionalidade`.

    ```bash
    git checkout -b feature/adicionar-funcionalidade
    ```

2.  Crie um arquivo de exemplo e commite-o.

    ```bash
    echo "def nova_funcao():" >> script.py
    echo "    pass" >> script.py
    git add script.py
    git commit -m "feat: Adiciona nova funcionalidade"
    ```

3.  Volte para a branch principal (`main`) e mescle as alterações.

    ```bash
    git checkout main
    git merge feature/adicionar-funcionalidade
    ```

4.  Remova a branch de funcionalidade.

    ```bash
    git branch -d feature/adicionar-funcionalidade
    ```

-----

## 3\. Configurando Git Hooks com Python

Agora, vamos criar um script para garantir que nenhuma informação sensível seja commitada.

### 3.1. Criando o Script do Hook

1.  Navegue até a pasta `.git/hooks` do seu projeto.

    ```bash
    cd .git/hooks
    ```

2.  Crie o arquivo do hook com o nome `pre-commit`.

    ```bash
    touch pre-commit
    ```

3.  Abra o arquivo `pre-commit` e adicione o seguinte código Python.

    ```python
      #!/usr/bin/env python3
      import subprocess
      import sys
      import re
      
      # Padrões para detectar informações sensíveis.
      # A regex foi aprimorada para focar em variáveis com valores literais,
      # ignorando as que chamam 'os.getenv' ou 'load_dotenv'.
      PADROES_SENSIVEIS = [
          re.compile(r'(api_key|token|password|secret|db_password)\s*=\s*[\'"][^\'"]+[\'"]', flags=re.IGNORECASE),
          re.compile(r'authorization: Bearer [^\s]+', flags=re.IGNORECASE),
          re.compile(r'texto proibido', flags=re.IGNORECASE)
      ]
      def verificar_arquivos_em_staging():
          """Verifica se há informações sensíveis nos arquivos na área de staging."""
      
      arquivos_em_staging = subprocess.getoutput("git diff --cached --name-only").splitlines()
  
  
      erros = []
  
      for arquivo in arquivos_em_staging:
          try:
              with open(arquivo, 'r', encoding='utf-8') as f:
                  conteudo = f.read()
  
                  for padrao in PADROES_SENSIVEIS:
                      if padrao.search(conteudo):
                          erros.append(f"❌ Erro: Padrão sensível '{padrao.pattern}' encontrado no arquivo '{arquivo}'.")
                          break
          except UnicodeDecodeError:
              continue
          except FileNotFoundError:
              continue
          except Exception as e:
              print(f"Aviso: Não foi possível ler o arquivo '{arquivo}'. Detalhes: {e}", file=sys.stderr)
  
      return erros
  
      if __name__ == "__main__":
          erros_encontrados = verificar_arquivos_em_staging()
          if erros_encontrados:
              print("\n🚨 O commit foi abortado devido aos seguintes problemas de segurança:")
              for erro in erros_encontrados:
                  print(f"   {erro}")
              print("\nPor favor, corrija os problemas e tente novamente.")
              sys.exit(1)
          else:
              print("\n✅ Verificação de pré-commit concluída. Nenhuma informação sensível detectada.")
              sys.exit(0)
    ```

4.  **Torne o script executável**.

    ```bash
    chmod +x pre-commit
    ```

-----

## 4\. Testando o Cenário de Segurança e Implementando a Solução

Agora, vamos simular o problema e, em seguida, aplicar a solução correta.

### 4.1. Testando o Hook

1.  Volte para a raiz do seu projeto.

    ```bash
    cd ../..
    ```

2.  Crie um arquivo de configuração, `config.py`, e adicione uma variável sensível.

    ```python
    # Este é um arquivo de configuração
    DB_PASSWORD = "minha_senha_super_secreta"
    API_KEY = "minha_chave_de_teste"
    ```

3.  Tente commitar o arquivo.

    ```bash
    git add config.py
    git commit -m "feat: Adiciona credenciais de teste"
    ```

O Git executará o seu script `pre-commit`, que detectará as credenciais e as palavras "teste" e "senha", exibirá uma mensagem de erro e **abortará o commit**.

### 4.2. Solucionando o Problema de Segurança

A maneira correta de lidar com informações sensíveis é movê-las para um arquivo de variáveis de ambiente (`.env`) e garantir que esse arquivo nunca seja enviado para o Git.

1.  **Crie o arquivo `.env`**.

    ```bash
    touch .env
    ```

2.  **Mova as credenciais** de `config.py` para o arquivo `.env`.

    ```bash
    echo "DB_PASSWORD=minha_senha_super_secreta" >> .env
    echo "API_KEY=minha_chave_de_teste" >> .env
    ```

3.  **Adicione o `.env` ao `.gitignore`**. Isso instrui o Git a ignorar este arquivo, garantindo que ele não será incluído nos seus commits.

    ```bash
    echo ".env" >> .gitignore
    ```

4.  **Instale a biblioteca `python-dotenv`** para que o seu código possa ler as variáveis.

    ```bash
    pip install python-dotenv
    ```

5.  **Atualize o `config.py`** para que ele carregue as variáveis do ambiente e não as armazene diretamente no código.

    ```python
    # config.py

    import os
    from dotenv import load_dotenv

    # Carrega as variáveis do arquivo .env
    # A função load_dotenv() procura o arquivo .env na raiz do projeto e carrega suas variáveis para o ambiente.
    load_dotenv()

    # Agora, você pode acessar as variáveis de ambiente como se fossem variáveis normais do sistema
    DB_PASSWORD = os.getenv("DB_PASSWORD")
    API_KEY = os.getenv("API_KEY")

    # Exemplo de uso
    print(f"A senha do banco de dados é: {DB_PASSWORD}")
    print(f"A chave da API é: {API_KEY}")
    ```

6.  **Remova as credenciais** que estavam no `config.py`.

7.  **Adicione e commite as alterações**. Desta vez, o commit passará, pois o `config.py` não contém mais as credenciais e o `.env` está sendo ignorado pelo Git.

    ```bash
    git add .gitignore config.py .env
    git status # Verifique se o .env está na lista de "ignored"
    git commit -m "feat: Move credenciais para o .env e ignora o arquivo"
    ```

Com este fluxo, você aprende a prevenir falhas de segurança e a implementar a solução ideal de forma automatizada.
