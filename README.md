1. Estrutura da Rede e Validação Inicial

A topologia do laboratório foi montada no Kathará integrando as seguintes redes e dispositivos:

  Internet / Roteador de Borda (r0): Responsável por simular a conexão com a rede externa e realizar a tradução de endereços (NAT).

  Firewall (fw): Atua como o ponto central de controle e filtragem entre todas as redes.

  Rede Interna (LAN): Contém as estações de trabalho pc1 e pc2.

  Zona Desmilitarizada (DMZ): Contém os servidores web (HTTP) e dns (DNS).

   Rede de Gerência (ADM): Contém a estação adm.

Teste de Conectividade Inicial (Sem Bloqueios)

Antes de ativar as regras do firewall, validamos se o roteamento e a comunicação básica estavam funcionando:

  LAN para Internet: O pc1 conseguiu navegar e pingar endereços externos.

  LAN para DMZ: O pc1 conseguiu acessar a página web do servidor web e consultar o servidor dns.

  Encaminhamento: O firewall fw repassou os pacotes entre as sub-redes sem restrições nesta etapa inicial.

2. Política de Segurança do Firewall (Perímetro)

Aplicamos no firewall a política de Bloqueio por Padrão (Default Deny), onde todo o tráfego é proibido, exceto o que for explicitamente liberado.

Utilizamos filtragem Stateful (com controle de estado), permitindo que o firewall reconheça automaticamente o tráfego de retorno de conexões legítimas.
Regras Implementadas

  Conexões Existentes: Permite a resposta de conexões que já foram iniciadas de forma autorizada.

  LAN → Internet: Liberado para os computadores da rede interna navegarem na web.

  LAN → DMZ: Liberado apenas para acesso ao Servidor Web (porta 80) e Servidor DNS (porta 53).

  Internet → DMZ: Liberado apenas para acesso público ao Servidor Web (porta 80).

  Internet → LAN: Totalmente bloqueado.

  DMZ → LAN: Totalmente bloqueado para novas conexões originadas na DMZ.

3. Experimentos em Diferentes Camadas (L2 a L7)
L2 — Camada de Enlace: Bloqueio por Endereço MAC

    O que foi feito: Simulamos que o pc2 foi infectado por um malware. Adicionamos uma regra no firewall para descartar qualquer pacote vindo do endereço físico (MAC) da placa de rede do pc2.

    Resultado: O pc1 continuou acessando os serviços normalmente, enquanto o pc2 teve todo o seu acesso cortado.

    Explicação: O endereço MAC só existe dentro da própria rede local. Quando um pacote passa por um roteador a caminho da Internet, o MAC original é trocado pelo MAC do roteador. O firewall só conseguiu bloquear o pc2 pelo MAC porque ambos estão conectados diretamente na mesma rede local.

L3 — Camada de Rede: ICMP e Bloqueio de IP
Experimento A: Bloqueio do Ping (ICMP)

  O que foi feito: Bloqueamos o protocolo ICMP no firewall e monitoramos a chegada de pacotes no servidor web usando a ferramenta de captura tcpdump.

  Resultado: O comando ping vindo da LAN falhou e os pacotes nem sequer chegaram ao servidor web. Porém, a navegação no site via curl continuou funcionando normalmente.

Experimento B: Bloqueio por IP de Destino

   O que foi feito: Criamos uma regra bloqueando o acesso a um endereço IP externo específico.

  Resultado: A LAN deixou de acessar aquele IP, mas continuou acessando outros destinos na Internet.

  Explicação: Bloquear apenas o IP não é uma boa solução para proibir um site. Sites grandes usam vários IPs diferentes, usam redes de distribuição de conteúdo (CDNs) com IPs que mudam constantemente, e um mesmo IP pode hospedar centenas de sites diferentes ao mesmo tempo.

L4 — Camada de Transporte: Bloqueio de Serviços (Portas)

  O que foi feito: Simulamos o bloqueio de programas P2P (como BitTorrent) fechando a faixa de portas padrão (6881 a 6889) no firewall.

  Resultado: As tentativas de conexão nessa faixa de portas foram recusadas.

  Explicação: Bloquear portas fixas não garante que o BitTorrent pare de funcionar. Aplicativos P2P modernos conseguem trocar de porta automaticamente, usar portas dinâmicas, criptografar o tráfego ou se disfarçar dentro de portas padrão como a 80 (HTTP) ou 443 (HTTPS).

L7 — Camada de Aplicação: Inspeção Avançada (WAF / Proxy)

  Conceito Escolhido: WAF (Firewall de Aplicação Web) e Proxy L7.

  Como Funciona: Diferente dos firewalls comuns que olham apenas IP e porta, o WAF analisa o conteúdo real da mensagem (o texto da requisição HTTP).

  O que ele permite fazer:

  Liberar o acesso à página pública de um site (/public) e bloquear a página de administração (/admin).
        
  Bloquear sites pelo nome do domínio e não pelo IP.

  Identificar qual aplicativo está rodando de verdade, bloqueando o BitTorrent mesmo se ele tentar usar a porta do navegador.

4. Defesa em Profundidade (Defense in Depth)
Cenário: O Servidor Web da DMZ foi Invadido

Se um atacante conseguir invadir e assumir o controle do servidor web na DMZ, ele não conseguirá acessar diretamente os computadores da rede interna (pc1 e pc2).
Camadas de Proteção que Impedem o Ataque

  Regra de Estado do Firewall: O firewall bloqueia qualquer nova conexão que tente sair da DMZ em direção à LAN.

  Segmentação de Rede: A LAN e a DMZ estão em redes isoladas. O atacante não consegue "escutar" a rede interna nem fazer ataques de rede local (como ARP Spoofing).

  Firewall Local nos Computadores: Cada computador na LAN pode ter seu próprio firewall ativado para barrar conexões não autorizadas.

  Restrição de Privilégios no Servidor: O servidor Web deve rodar com um usuário limitado, impedindo que o atacante use a máquina para varrer a rede.

Conclusão: O princípio da Defesa em Profundidade garante que a segurança não dependa de uma única barreira. Mesmo com a invasão do servidor na DMZ, as outras camadas impedem que o ataque avance para a rede interna.
