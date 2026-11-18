# Revisão de segurança

A revisão parte do código aberto do Wireless Battery no commit upstream
`30419ed4e29ffd1b1337eccf367f2381faebd592` e do ZIP portátil oficial da versão
`v1.0.8`. São referências diferentes. O executável oficial e o do fork não
foram comparados byte a byte.

## Superfícies revisadas

- **Windows e periféricos.** O aplicativo enumera dispositivos com SetupAPI e
  consulta a bateria via HID, Bluetooth e protocolos específicos por fabricante.
  Alguns drivers enviam relatórios HID para solicitar o estado da bateria. A
  análise do código não confirma o comportamento de todos os periféricos.
- **Execução de terceiros.** O HeadsetControl é opcional. A versão reconstruída
  procura o auxiliar apenas ao lado do aplicativo, rejeita um arquivo marcado
  como ponto de nova análise e fornece o caminho completo a CreateProcess.
  A leitura da saída tem limites de tempo e tamanho. A confiança na pasta de
  instalação continua necessária. O script de empacotamento fixa a versão
  4.3.0 e verifica o SHA-256 do auxiliar antes de incluí-lo.
- **Rede.** O envio pelo botão do formulário de suporte faz um POST HTTPS para
  `krisgfx.com/tools/api/requests` com modelo, tipo, notas e nome opcional.
  A versão reconstruída limita a resposta a 16 KiB e carrega as duas DLLs
  usadas dinamicamente pelo formulário da pasta de sistema. O aplicativo
  também pode abrir links de apoio no navegador por ação do usuário.
- **Instalação e persistência.** O inicializador usa a chave Run do usuário.
  O instalador reconstruído deixa a execução inicial e a inicialização
  automática desmarcadas. O instalador oferece instalação por usuário e
  instalação para todos os usuários.
- **Linux.** A regra udev fornecida concede acesso aos dispositivos hidraw para
  a sessão local ativa de forma ampla. A regra foi inspecionada como texto.
- **CI e distribuição.** Os workflows usam ações GitHub fixadas por commit.
  O scanner de PR analisa o diff como texto. Ele usa heurísticas e não certifica
  a segurança de uma alteração. O processo de empacotamento também depende do
  compilador, do Inno Setup e dos artefatos obtidos de terceiros.

## Artefato oficial e limites

O REA identificou o `WirelessBattery.exe` oficial como PE x86-64, com SHA-256
`ad3bca7af7b56262e3a30e6f83ab1f9fc937b286926323fa6ffae1a5d561ddba`.
A inspeção dos cabeçalhos e da tabela de importação com objdump encontrou
`DYNAMIC_BASE`, `NX_COMPAT` e as bibliotecas WinHTTP, Winsock, HID e SetupAPI.
O executável e o auxiliar do ZIP estavam sem assinatura Authenticode. O hash
do auxiliar coincidia com o valor fixado no script de empacotamento.
O inventário estático do ZIP pelo REA não encontrou contradições de integridade
(evidência `ev_a4727bb488ec60e6922a9c20b15c736f23ae915b7fdf0c88c1ab5045350fa5a5`).
O instalador oficial também estava sem assinatura Authenticode. O Microsoft
Defender concluiu varreduras locais do ZIP extraído e do instalador oficial
sem registrar detecções.

O executável reconstruído a partir do commit `48bdc51` compilou com
MinGW-w64, e todos os testes do alvo `make test` foram aprovados no Windows.
O SHA-256 do executável reconstruído é
`31cbd2872f279cc9ac40275dac251cdae35d015b50fccac826393f86dd93ccc3`.
A varredura desse arquivo no Defender foi concluída sem registrar detecções.

O provedor de descompilação do REA estava indisponível. A revisão foi estática.
Ela não inclui análise dinâmica em VM, reconstrução do pacote Inno Setup,
auditoria completa das dependências de terceiros ou testes com hardware
real. Os resultados não certificam ausência
de malware, ransomware ou vulnerabilidades.
