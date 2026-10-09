# Linux Network Security Lab
[English](README.md) | Português

Laboratório pessoal de redes e segurança em uma VM Debian 13. Configurei o UFW e acessei um servidor HTTP na VM pelo Windows para entender a relação entre serviço, porta e firewall.

## Ambiente
- **Host:** Windows 11.
- **VM:** Debian 13 no VirtualBox.
- **Rede:** inicialmente NAT; alterada para modo bridge durante o teste.
- **Ferramentas:** UFW, Python 3, `ip` e `ss`.

No modo bridge, a VM recebeu o endereço privado `192.168.18.247/24`. O `/24` indica o prefixo da rede, não uma porta. Esse endereço pertence ao meu ambiente e pode mudar.

## O que fiz

### 1. Verificação da rede e dos serviços
```bash
ip a
ip route
sudo ss -tulpn
```

Verifiquei endereços, rotas e sockets TCP/UDP locais. Identifiquei o Avahi nas linhas UDP e o CUPS na porta TCP 631, associado aos endereços de loopback `127.0.0.1` e `::1`.

O resultado do `ss` mostra sockets locais; sozinho, ele não confirma se um serviço está acessível de outra máquina.

### 2. Configuração do firewall
```bash
sudo apt update
sudo apt install ufw
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw status verbose
```

Confirmei o firewall ativo, com entrada negada e saída permitida por padrão. Essas políticas se aplicam à VM. Respostas às conexões iniciadas por ela continuam permitidas.

Também pratiquei adicionar e remover uma exceção:
```bash
sudo ufw allow 8080/tcp
sudo ufw delete allow 8080/tcp
```

### 3. Teste de acesso pelo Windows
Dentro de uma pasta dedicada ao teste, iniciei um servidor HTTP:
```bash
mkdir lab-web
cd lab-web
python3 -m http.server 8080
```

O `-m` executa o módulo `http.server` do Python, que disponibiliza o conteúdo da pasta pela porta 8080.

Liberei a conexão no UFW:
```bash
sudo ufw allow 8080/tcp
```

Com o servidor rodando, acessei pelo navegador do Windows:
```text
http://192.168.18.247:8080
```

O navegador exibiu **Directory listing for /**, confirmando o acesso ao servidor na VM.

## Resultados e aprendizado
- Configurei e conferi as políticas do UFW.
- Adicionei e removi uma regra para TCP/8080.
- Consegui acessar o servidor da VM a partir do Windows.
- Durante o teste, interrompi o servidor para executar outro comando. Precisei iniciá-lo novamente: liberar uma porta no firewall não cria nem mantém um serviço rodando.
