# Exercícios Práticos com Docker

Aqui estão os exercícios Docker formatados para facilitar o acompanhamento. Se voce quiser ver o conteudo em video, voce pode acessar a video aula neste link:

[Containers vs Docker: Entenda Tudo Sobre Virtualização Leve!](https://www.youtube.com/watch?v=wUsAQmk8Y3M&ab_channel=Jo%C3%A3oPedroSchmidtCordeiro)

# Exercícios:

1. [Exercício 1: Aplicação Python Básica](#exercício-1-aplicação-python-básica)
2. [Exercício 2: API Flask com Versionamento](#exercício-2-api-flask-com-versionamento)
3. [Exercício 3: Rodando n8n com Docker](#exercício-3-rodando-n8n-com-docker)
4. [Exercício 4: Comunicação Cliente-Servidor entre Containers](#exercício-4-comunicação-cliente-servidor-entre-containers)

## Exercício 1: Aplicação Python Básica

**Objetivo:**
Rodar um container com uma aplicação básica Python usando Docker.

**Passos:**

1. **`Crie o arquivo app.py`** com o seguinte conteúdo:
    
    ```
    print("Olá, mundo com Docker!")
    
    ```
    
2. **`Crie o arquivo Dockerfile`** com o seguinte conteúdo:
    
    ```
    # Usa a imagem base oficial do Python 3.10 slim
    FROM python:3.10-slim
    
    # Copia o script Python para o diretório /app dentro do container
    COPY app.py /app/app.py
    
    # Define o diretório de trabalho para /app
    WORKDIR /app
    
    # Define o comando padrão para executar quando o container iniciar
    CMD ["python", "app.py"]
    
    ```
    
3. **Execute os comandos no seu terminal:**
    - Para construir a imagem Docker:
        
        ```
        docker build -t hello-docker .
        
        ```
        
    - Para rodar o container a partir da imagem criada (o `-rm` remove o container após a execução):
        
        ```
        docker run --rm hello-docker
        
        ```
        
    - Você deverá ver a saída: `Olá, mundo com Docker!`

**Conceitos:**

- **Empacotamento com imagem:** O `Dockerfile` define como sua aplicação é empacotada em uma imagem Docker.
- **Isolamento de ambiente:** O container roda em um ambiente isolado, com suas próprias dependências (neste caso, Python 3.10).
- **Independência do host:** A aplicação roda da mesma forma, independentemente do sistema operacional do seu computador (host), desde que o Docker esteja instalado.

## Exercício 2: API Flask com Versionamento

**Objetivo:**
Criar uma API web simples com Flask, gerar uma imagem Docker para ela e aplicar versionamento usando tags.

**Arquivos:**

1. **`app.py`** (Versão 1.0):
    
    ```
    from flask import Flask
    
    # Cria uma instância da aplicação Flask
    app = Flask(__name__)
    
    # Define uma rota para a raiz ("/") que retorna uma mensagem
    @app.route("/")
    def home():
        return "Versão 1.0 da API Dockerizada!"
    
    # (O comando CMD no Dockerfile iniciará este servidor)
    
    ```
    
2. **`requirements.txt`**:
    
    ```
    Flask==2.2.2
    
    ```
    
3. **`Dockerfile`**:
    
    ```
    # Usa a imagem base oficial do Python 3.10 slim
    FROM python:3.10-slim
    
    # Define o diretório de trabalho dentro do container
    WORKDIR /app
    
    # Copia o arquivo de dependências primeiro (aproveita o cache do Docker)
    COPY requirements.txt .
    
    # Instala as dependências Python listadas no requirements.txt
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Copia o restante dos arquivos da aplicação (app.py)
    COPY . .
    
    # Expõe a porta 5000, que o Flask usa por padrão
    EXPOSE 5000
    
    # Define o comando para rodar a aplicação Flask quando o container iniciar
    # flask run --host=0.0.0.0 permite que a API seja acessível de fora do container
    CMD ["flask", "run", "--host=0.0.0.0"]
    
    ```
    
    `*(Nota: Atualizei o CMD para flask run --host=0.0.0.0 que é a forma recomendada de rodar o servidor de desenvolvimento Flask, tornando-o acessível externamente ao container).*`
    

**Comandos:**

1. **Construir a versão 1.0 da imagem:**
    
    ```
    docker build -t minha-api:1.0 .
    
    ```
    
2. **Rodar a versão 1.0:**
    
    ```
    docker run -d -p 5000:5000 --name api-v1 minha-api:1.0
    
    ```
    
    - Acesse `http://localhost:5000` no seu navegador. Você verá "Versão 1.0 da API Dockerizada!". Use `docker stop api-v1 && docker rm api-v1` para parar e remover o container. (Adicionado `d` e `-name` para melhor gerenciamento).
3. **`Atualizar app.py para a Versão 2.0:`**
    - Modifique o arquivo `app.py` para:
        
        ```
        from flask import Flask
        app = Flask(__name__)
        @app.route("/")
        def home():
            return "Esta é a incrível Versão 2.0 da API!" # Mensagem atualizada
        
        ```
        
4. **Construir a versão 2.0 da imagem:**
    
    ```
    docker build -t minha-api:2.0 .
    
    ```
    
5. **Rodar a versão 2.0:**
    
    ```
    docker run -d -p 5000:5000 --name api-v2 minha-api:2.0
    
    ```
    
    - Acesse `http://localhost:5000`. Agora você verá "Esta é a incrível Versão 2.0 da API!". Use `docker stop api-v2 && docker rm api-v2` para parar e remover.
6. **Reverter para a versão anterior (Rollback):**
    
    ```
    docker run -d -p 5000:5000 --name api-v1-rollback minha-api:1.0
    
    ```
    
    - Acesse `http://localhost:5000` novamente. Você voltará a ver a mensagem da versão 1.0. Use `docker stop api-v1-rollback && docker rm api-v1-rollback` para limpar.

**Conceitos:**

- **Controle de versões (tags):** Usar tags (`:1.0`, `:2.0`) permite gerenciar diferentes versões da sua imagem.
- **Camadas de imagem:** Cada instrução no `Dockerfile` (como `COPY`, `RUN`) cria uma camada na imagem. O Docker reutiliza camadas inalteradas (cache), acelerando builds subsequentes.
- **Portabilidade:** A imagem Docker contém tudo que a API precisa para rodar, tornando-a portátil entre diferentes ambientes.
- **Reversão de versão (rollback):** É fácil voltar para uma versão anterior simplesmente rodando a imagem com a tag correspondente.

## Exercício 3: Rodando n8n com Docker

**Objetivo:**
Rodar a ferramenta de automação de fluxo de trabalho n8n usando Docker, garantindo a persistência dos dados.

**Passos:**

1. **Crie um diretório local para persistir os dados do n8n:**
    - Este comando cria o diretório `n8n-data` no seu diretório atual, se ele não existir.
    
    ```
    mkdir -p n8n-data
    
    ```
    
2. **Execute o container do n8n:**
    - Este comando baixa a imagem `n8nio/n8n` (se ainda não existir) e inicia um container.
    
    ```
    docker run -it --rm \\
      --name n8n \\
      -p 5678:5678 \\
      -v "$(pwd)/n8n-data:/home/node/.n8n" \\
      -e N8N_BASIC_AUTH_ACTIVE=true \\
      -e N8N_BASIC_AUTH_USER=admin \\
      -e N8N_BASIC_AUTH_PASSWORD=admin123 \\
      n8nio/n8n
    
    ```
    
    - **Explicação dos parâmetros:**
        - `it`: Modo interativo, conecta seu terminal ao container.
        - `-rm`: Remove o container quando ele é parado.
        - `-name n8n`: Dá um nome fácil de identificar ao container.
        - `p 5678:5678`: Mapeia a porta 5678 do seu computador para a porta 5678 do container (onde o n8n roda).
        - `v "$(pwd)/n8n-data:/home/node/.n8n"`: Monta o diretório local `n8n-data` (que você criou) no diretório `/home/node/.n8n` dentro do container. É aqui que o n8n guarda seus dados (workflows, credenciais). Isso garante que seus dados persistam mesmo que o container seja parado e recriado. `$(pwd)` garante que o caminho absoluto do diretório atual seja usado.
        - `e N8N_...`: Define variáveis de ambiente dentro do container para configurar a autenticação básica.
3. **Acesse o n8n:**
    - Abra seu navegador e vá para `http://localhost:5678`.
    - Use as credenciais definidas nas variáveis de ambiente:
        - **Usuário:** `admin`
        - **Senha:** `admin123`

**Conceitos:**

- **`Implantação rápida com docker run:`** Permite iniciar aplicações complexas como o n8n com um único comando.
- **`Volume persistente (-v):`** Garante que os dados gerados pela aplicação (workflows do n8n) não sejam perdidos quando o container for parado ou removido, pois são armazenados no seu sistema de arquivos local.
- **`Isolamento e segurança via variáveis de ambiente (-e):`** Configurações sensíveis (como credenciais) podem ser passadas de forma segura para o container sem precisar colocá-las diretamente na imagem.
- **Aplicação útil no dia a dia:** O n8n é uma ferramenta poderosa para automação que pode ser facilmente testada e utilizada com Docker.

## Exercício 4: Comunicação Cliente-Servidor entre Containers

**Objetivo:**
Demonstrar como dois containers Docker (um servidor web simples e um cliente) podem se comunicar usando uma rede Docker customizada.

**Arquivos:**

1. **`server.py`**: (Crie este arquivo)
    
    ```
    # server.py
    from http.server import BaseHTTPRequestHandler, HTTPServer
    import time
    
    hostName = "0.0.0.0" # Importante para escutar em todas as interfaces de rede do container
    serverPort = 8080
    
    class MyServer(BaseHTTPRequestHandler):
        def do_GET(self):
            self.send_response(200)
            self.send_header("Content-type", "text/plain; charset=utf-8")
            self.end_headers()
            self.wfile.write(bytes("Olá do servidor Docker!", "utf-8"))
    
    if __name__ == "__main__":
        webServer = HTTPServer((hostName, serverPort), MyServer)
        print(f"Servidor iniciado em http://{hostName}:{serverPort}") # Log no container
    
        try:
            webServer.serve_forever()
        except KeyboardInterrupt:
            pass
    
        webServer.server_close()
        print("Servidor parado.")
    
    ```
    
2. **`Dockerfile.server`**: (Crie este arquivo com nome diferente do anterior)
    
    ```
    # Dockerfile.server
    FROM python:3.10-slim
    WORKDIR /app
    COPY server.py .
    # Expõe a porta que o servidor usa
    EXPOSE 8080
    # Comando para iniciar o servidor
    CMD ["python", "server.py"]
    
    ```
    
3. **`client.py`**: (Crie este arquivo)
    
    ```
    # client.py
    import requests
    import time
    import os
    
    # O nome 'meu-servidor' será resolvido pelo DNS interno do Docker
    # quando ambos os containers estiverem na mesma rede customizada.
    server_url = "http://meu-servidor:8080"
    max_retries = 5
    wait_seconds = 3 # Aumentar um pouco a espera inicial
    
    print(f"Cliente tentando conectar ao servidor em {server_url}...")
    
    # Tenta conectar algumas vezes, pois o servidor pode demorar um pouco para iniciar
    for attempt in range(max_retries):
        try:
            print(f"Tentativa {attempt + 1}/{max_retries}...")
            response = requests.get(server_url, timeout=5) # Timeout para a requisição
            response.raise_for_status() # Verifica se houve erro HTTP (4xx ou 5xx)
    
            print(f"--- SUCESSO! ---")
            print(f"Status Code: {response.status_code}")
            print(f"Resposta do servidor: '{response.text}'")
            print(f"----------------")
            exit(0) # Sucesso, termina o cliente
    
        except requests.exceptions.ConnectionError as e:
            print(f"Falha na conexão: {e}")
            if attempt < max_retries - 1:
                print(f"Servidor ainda não disponível? Tentando novamente em {wait_seconds}s...")
                time.sleep(wait_seconds)
            else:
                print("Número máximo de tentativas atingido. O servidor está rodando e na mesma rede?")
                exit(1) # Erro, termina o cliente
    
        except requests.exceptions.RequestException as e:
            # Captura outros erros do requests (timeout, etc.)
            print(f"Ocorreu um erro na requisição: {e}")
            exit(1) # Erro, termina o cliente
    
    # Se sair do loop sem sucesso (embora o exit() deva ter sido chamado)
    print("Não foi possível conectar ao servidor após múltiplas tentativas.")
    exit(1)
    
    ```
    
4. **`requirements.txt`** (para o cliente): (Crie este arquivo)
    
    ```
    requests
    
    ```
    
5. **`Dockerfile.client`**: (Crie este arquivo com nome diferente)
    
    ```
    # Dockerfile.client
    FROM python:3.10-slim
    WORKDIR /app
    COPY requirements.txt .
    RUN pip install --no-cache-dir -r requirements.txt
    COPY client.py .
    # Não precisa de EXPOSE, pois é um cliente
    # O CMD executa o script cliente
    CMD ["python", "client.py"]
    
    ```
    

**Passos:**

1. **Crie os 5 arquivos** listados acima no mesmo diretório.
2. **Crie uma rede Docker customizada:**
    - Containers na rede padrão (`bridge`) precisam usar IPs para se comunicar. Uma rede customizada permite que usem os nomes dos containers (DNS).
    
    ```
    docker network create minha-rede
    
    ```
    
3. **Construa as imagens Docker:**
    - Construa a imagem do servidor:
        
        ```
        docker build -t meu-servidor-img -f Dockerfile.server .
        
        ```
        
    - Construa a imagem do cliente:
        
        ```
        docker build -t meu-cliente-img -f Dockerfile.client .
        
        ```
        
4. **Execute o container do Servidor:**
    - Rode o servidor em background (`d`) na rede criada e dê um nome a ele (`-name meu-servidor`). Este nome será usado pelo cliente.
    
    ```
    docker run -d --network minha-rede --name meu-servidor meu-servidor-img
    
    ```
    
    - Você pode verificar se ele está rodando com `docker ps` e ver seus logs com `docker logs meu-servidor`.
5. **Execute o container do Cliente:**
    - Rode o cliente na mesma rede. Ele tentará conectar ao `http://meu-servidor:8080`. Use `-rm` para que o container seja removido após a execução.

    ```
    docker run --rm --network minha-rede --name meu-cliente meu-cliente-img
    
    ```
    
    - **Observe a saída:** Você deverá ver as tentativas de conexão do cliente e, finalmente, a mensagem de sucesso com a resposta "Olá do servidor Docker!".
6. **Limpeza (Opcional):**
    - Pare e remova o container do servidor:
        
        ```
        docker stop meu-servidor
        docker rm meu-servidor
        
        ```
        
    - Remova a rede:
        
        ```
        docker network rm minha-rede
        
        ```
        
    - Remova as imagens (se desejar):
        
        ```
        docker rmi meu-cliente-img meu-servidor-img
        
        ```
        

**Conceitos:**

- **Comunicação entre Containers:** Como aplicações em containers diferentes podem interagir.
- **Redes Docker:** A importância das redes customizadas (tipo `bridge`) para permitir a comunicação por nome de serviço.
- **Resolução de Nomes (DNS Docker):** O Docker fornece um DNS interno para containers na mesma rede customizada, permitindo que `http://meu-servidor` seja resolvido para o IP correto do container do servidor.
- **Arquitetura Cliente-Servidor:** Um padrão comum onde um cliente requisita serviços de um servidor.
- **`Execução em Background (-d):`** Como rodar containers sem prender o terminal.
- **`Logs de Container (docker logs):`** Como visualizar a saída de um container rodando em background.