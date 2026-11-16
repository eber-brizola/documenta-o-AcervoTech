# documenta-o-AcervoTech
# AcervoTech — Portal de Transparência Escolar

O **AcervoTech** é uma aplicação web para organizar e disponibilizar documentos escolares. O portal público permite consultar categorias, pesquisar arquivos e realizar visualizações e downloads. O painel administrativo reúne as ferramentas de publicação e gerenciamento do acervo.

O projeto foi desenvolvido no contexto do curso técnico em Informática e do Trabalho de Conclusão de Curso, com atenção à aplicação de controles de segurança em sistemas web. Esta documentação orienta o uso, a instalação e a continuidade do desenvolvimento.

**Instituição:** Colégio Estadual Dr. Dorvalino Luciano de Souza — Cerro Grande/RS.  
**Identificação na interface:** CEEDO.  
**Última atualização da documentação:** 16 de novembro de 2026.

Material complementar: [Lista de siglas e apoio ao seminário](docs/SIGLAS.md).

## Sumário

- [1. Visão geral](#1-visão-geral)
- [2. Perfis de acesso](#2-perfis-de-acesso)
- [3. Tecnologias](#3-tecnologias)
- [4. Páginas públicas](#4-páginas-públicas)
- [5. Painel: documentos e categorias](#5-painel-documentos-e-categorias)
- [6. Operadores](#6-operadores)
- [7. Autenticação e segurança](#7-autenticação-e-segurança)
- [8. Banco de dados](#8-banco-de-dados)
- [9. Instalação e publicação](#9-instalação-e-publicação)
- [10. Manutenção e organização](#10-manutenção-e-organização)
- [11. Roteiro de testes](#11-roteiro-de-testes)
- [12. Continuidade do projeto](#12-continuidade-do-projeto)

## 1. Visão geral

O sistema possui duas áreas: o portal público, voltado à consulta dos documentos, e o painel administrativo, acessível mediante autenticação. Os arquivos publicados ficam disponíveis aos visitantes; a autenticação protege o gerenciamento do acervo.

### Funcionalidades

- Consulta de categorias e documentos recentes.
- Pesquisa por nome, filtros por categoria e tipo de arquivo, ordenação e paginação.
- Visualização e download de PDF, JPG e PNG.
- Cadastro, edição e exclusão de documentos.
- Cadastro e edição de categorias, com descrição e seleção de ícones.
- Administração de contas com distinção entre os níveis `adm` e `root`.
- Página Ajuda com orientações para utilização do painel.

### Escopo e validação

Esta documentação descreve a versão de código analisada em novembro de 2026. A configuração do banco e do servidor deve ser validada em cada instalação. O roteiro de testes apresenta resultados esperados; não representa um relatório de testes executados nem certificação de segurança. Melhorias ainda não confirmadas na versão em uso estão indicadas na seção de continuidade.

## 2. Perfis de acesso

| Perfil | O que faz |
| --- | --- |
| Visitante | Consulta categorias, pesquisa documentos, visualiza e baixa arquivos públicos. |
| Administrador (`adm`) | Entra no painel e cadastra, edita e exclui documentos e categorias. |
| Root (`root`) | Usa o painel e gerencia administradores: cadastro, promoção, ativação, desativação e exclusão, conforme as regras dos respectivos arquivos. |

A proteção depende da verificação feita no servidor. Esconder o item Operadores na sidebar melhora a interface, mas não substitui `verifica_root.php` nas páginas e ações protegidas. A transferência exclusiva do cargo e o rebaixamento de root não fazem parte do fluxo documentado nesta versão.

## 3. Tecnologias

- PHP: processa formulários, sessões, permissões e entrega de arquivos.
- MySQL/MariaDB com MySQLi: armazena categorias, documentos, operadores e contadores de tentativas.
- HTML e CSS: estrutura e aparência das páginas.
- JavaScript: interação do seletor de arquivos e outros comportamentos da interface.
- Bootstrap e Bootstrap Icons: componentes visuais e ícones. O código inclui recursos por CDN, que dependem de acesso à internet.
- Apache/XAMPP: ambiente local usado pelo autor. Hostinger: hospedagem publicada, cuja configuração deve ser conferida separadamente.

As extensões PHP usadas incluem `mysqli`, `mbstring` e `fileinfo`. O uso de `mysqli_stmt_get_result()` também exige suporte apropriado do driver, normalmente mysqlnd. Escolha uma versão PHP com suporte de segurança e teste a aplicação antes de atualizar produção.

## 4. Páginas públicas

| Arquivo | Responsabilidade |
| --- | --- |
| `index.php` | Página inicial: apresentação, categorias e os dez documentos mais recentes. |
| `categorias.php` | Lista as categorias disponíveis no portal público. |
| `categoria.php` | Mostra os documentos da categoria identificada na URL. |
| `documentos.php` | Busca por nome, filtros por categoria e tipo, ordenação e paginação. |
| `visualizar.php` | Solicita a entrega do documento para visualização no navegador. |
| `download.php` | Solicita a entrega do arquivo como download. |
| `includes/servir_documento.php` | Consulta o documento pelo ID, confere caminho e tipo e envia seu conteúdo. |
| `includes/icones_categorias.php` | Lista de classes de ícones aceitas pelo sistema. |
| `includes/navbar.php` e `includes/footer.php` | Navegação e rodapé compartilhados do portal. |

A pesquisa usa parâmetros nas consultas e uma lista fixa de ordenações permitidas. Os resultados são paginados, com 12 itens por página. Um filtro não deve ser incorporado livremente como trecho SQL.

## 5. Painel: documentos e categorias

| Arquivo em `painel/` | Responsabilidade |
| --- | --- |
| `index.php` | Dashboard e atalhos de administração. |
| `documentos.php` | Lista os documentos para administração. |
| `novo_documento.php` | Formulário de cadastro e seleção visual do arquivo. |
| `salvar_documento.php` | Valida os campos e o upload, guarda o arquivo e cria o registro. |
| `editar_documento.php` | Carrega os dados para edição. |
| `atualizar_documento.php` | Atualiza o documento e permite substituir o arquivo. |
| `excluir_documento.php` | Remove o registro e tenta limpar o arquivo sem referências. |
| `categorias.php` | Lista categorias e oferece seu cadastro. |
| `salvar_categoria.php` | Valida e cadastra a categoria e seu ícone. |
| `editar_categoria.php` | Carrega nome, descrição e ícone para edição. |
| `atualizar_categoria.php` | Salva as alterações da categoria. |
| `remover_categoria.php` | Exclui categoria somente quando não existem documentos associados. |
| `ajuda.php` | Manual de uso disponível para os administradores. |

### Publicar um documento

1. Entre no painel e abra **Adicionar documento**.
2. Informe nome e categoria.
3. Selecione PDF, JPG ou PNG, com conteúdo e até 10 MiB (10 × 1024 × 1024 bytes).
4. Confira o nome apresentado na área de upload.
5. Clique em **Salvar documento** e aguarde a confirmação.
6. Confira a visualização e o download no portal público.

A área verde apenas confirma a seleção no navegador. O upload acontece ao enviar o formulário. O servidor repete as validações, pois as verificações JavaScript podem ser contornadas.

Ao editar, deixar o campo de arquivo vazio mantém o arquivo anterior. A substituição recebe um nome aleatório novo. Após a confirmação no banco, o sistema tenta remover o arquivo antigo quando não existem outras referências a ele.

Banco e sistema de arquivos não constituem uma única transação. Em uma falha de conexão, o sistema pode preservar um arquivo para evitar apagar um documento cujo cadastro foi confirmado pelo banco sem que a aplicação tenha recebido a resposta. Confira a listagem antes de repetir a operação. Arquivos órfãos podem exigir conferência posterior.

### Categorias e ícones

A biblioteca Bootstrap Icons fornece os desenhos. A coluna `icone` guarda apenas a classe escolhida, por exemplo `bi-folder`. A lista de classes permitidas está em `includes/icones_categorias.php`. O painel e a página inicial utilizam essa seleção. Na versão-base analisada, a listagem pública `categorias.php` ainda associa ícones ao nome da categoria; sua uniformização deve ser conferida ao atualizar o projeto.

Nome de categoria: até 100 caracteres. Descrição: até 255 caracteres. Nome de documento: até 255 caracteres. Os limites de interface devem continuar acompanhados de validação no servidor.

## 6. Operadores

| Arquivo em `painel/` | Responsabilidade |
| --- | --- |
| `operadores.php` | Listagem e ações de gerenciamento, restritas a root. |
| `novo_operador.php` e `salvar_operador.php` | Formulário e criação de administrador. |
| `confirmar_promocao.php` e `promover_operador.php` | Confirmação e execução da promoção de administrador ativo a root. |
| `confirmar_acao_operador.php` e `alterar_operador.php` | Confirmação e execução de desativação, reativação ou exclusão de administrador. |

As ações sensíveis conferem novamente o autor no banco e solicitam sua senha. Os registros envolvidos são bloqueados durante a transação para reduzir conflitos entre operações simultâneas. O código impede usar essas operações para modificar a própria conta e restringe os destinatários aos perfis previstos.

A criação de administrador usa autenticação root e CSRF, mas não possui a mesma confirmação de senha das outras ações. Essa uniformização pode ser uma melhoria posterior. Não confundir promoção de administrador com transferência exclusiva do cargo: podem existir várias contas root.

## 7. Autenticação e segurança

| Arquivo | O que faz e qual problema reduz |
| --- | --- |
| `autentica.php` | Recebe o login, verifica senha e situação da conta e cria a sessão autenticada. |
| `painel/includes/sessao.php` | Configura e inicia a sessão, incluindo propriedades dos cookies. |
| `painel/includes/verifica_login.php` | Exige sessão válida e confere conta ativa, nível e versão da sessão no banco. |
| `painel/includes/verifica_root.php` | Exige perfil root, inclusive se alguém acessar a URL diretamente. |
| `painel/includes/csrf.php` | Gera e valida token para reduzir requisições de alteração forjadas de outro site. |
| `painel/includes/limite_root.php` | Registra no MySQL o limite compartilhado de confirmações root. |
| `painel/logout.php` | Encerra o acesso mediante a requisição prevista pelo formulário. |

### Controles observados no código

- **Senha com hash:** `password_verify()` compara a senha informada ao hash; a senha de usuário não deve ser armazenada em texto puro.
- **Tentativas de login:** até cinco tentativas por e-mail na janela de 15 minutos; o contador fica no banco. Há limpeza probabilística de registros antigos. Não existe limite por IP nesta versão.
- **Confirmações root:** contador persistente, até cinco confirmações por janela de 15 minutos. As confirmações permitidas são contadas, inclusive quando a operação é autorizada. Trocar de sessão não reinicia esse contador.
- **Renovação do identificador:** o login renova o ID da sessão para reduzir fixação de sessão.
- **Inatividade:** o painel encerra o acesso após 30 minutos sem requisições, na próxima requisição protegida. Uma página já aberta não muda sozinha por esse mecanismo.
- **Versão da sessão:** mudanças previstas nas contas incrementam a versão; uma sessão antiga deixa de ser aceita na próxima verificação.
- **Cookies:** HttpOnly, SameSite Lax e Secure quando o PHP detecta HTTPS. É necessário conferir essa detecção na hospedagem.
- **SQL parametrizado:** os valores do usuário são enviados separadamente da estrutura da consulta nas rotinas revisadas.
- **Saída HTML:** `htmlspecialchars()` é utilizado para exibir diversos valores sem interpretá-los como marcação. Cada nova saída de dados precisa manter o escape adequado ao seu contexto.
- **Upload:** tamanho, origem temporária e MIME são conferidos; o nome salvo é aleatório. Isso não equivale a antivírus ou garantia de conteúdo inofensivo.
- **Entrega de arquivos:** o caminho real deve estar dentro de uploads; o servidor envia apenas os tipos previstos, com cabeçalho `nosniff`.

CSRF, SQL injection e XSS são problemas diferentes. Token CSRF não valida conteúdo de arquivos; consultas preparadas não escapam HTML; esconder um botão não controla permissões.

## 8. Banco de dados

A estrutura abaixo foi identificada a partir das consultas PHP. O esquema SQL de instalação precisa acompanhar a versão do código. Tipos, índices e restrições devem ser conferidos no banco real; esta tabela não substitui o script de criação das tabelas.

| Tabela | Finalidade / campos observados |
| --- | --- |
| `categorias` | ID, nome, descrição e classe do ícone. |
| `documentos` | ID, nome, categoria, caminho do arquivo e data de upload. |
| `operadores` | ID, nome, e-mail, hash da senha, nível, ativo e versão de sessão. |
| `login_tentativas` | Chave derivada do e-mail, quantidade de tentativas e início da janela. |
| `root_confirmacoes` | Controle de confirmações por operador, com tentativas e início da janela. |

Cada documento aponta para uma categoria. O arquivo físico fica em uploads; o banco guarda o caminho. Confirme no banco real as chaves primárias, índices únicos, relações e o uso de mecanismo com transações, como InnoDB. A prevenção de duplicidade de e-mail depende também da restrição no banco. Não recrie tabelas apenas a partir deste resumo.

## 9. Instalação e publicação

### Preparar o ambiente local

No XAMPP para Linux, coloque o projeto em `/opt/lampp/htdocs/tcc/` e inicie Apache e MySQL pelo gerenciador do XAMPP. Depois, acesse:

- Portal público: `http://localhost/tcc/`
- Login administrativo: `http://localhost/tcc/login/`
- Painel: `http://localhost/tcc/painel/`

Se a pasta tiver outro nome, ajuste o endereço correspondente. O projeto precisa ser servido pelo Apache/PHP; abrir um arquivo PHP diretamente no navegador não executa o sistema.

Importe, pelo phpMyAdmin ou cliente MySQL, o SQL correspondente à versão do projeto. O arquivo de instalação do banco não foi validado junto desta documentação; obtenha-o com o responsável pelo projeto antes de preparar uma instalação nova. Não importe um banco de outra versão sobre dados de produção.

Ajuste host, usuário, senha e nome do banco em `painel/conexao.php`. Use uma conta existente da instalação para entrar. A preparação da primeira conta root deve ser feita pelo responsável técnico com hash de senha apropriado; esta documentação não fornece uma senha padrão nem pressupõe a existência de um instalador automático.

### Conferências para instalar ou atualizar

1. Faça backup do banco e dos uploads, juntos, antes de substituir uma versão.
2. Use uma cópia do projeto em ambiente local e importe o banco correspondente à mesma versão.
3. Configure `painel/conexao.php` com os dados daquele ambiente. Não publique senhas em README, prints ou repositórios.
4. Confira as extensões PHP, a configuração de erros e os limites `upload_max_filesize` e `post_max_size`. O segundo deve comportar o arquivo e os demais dados do formulário.
5. Garanta que o usuário do servidor possa escrever em uploads; evite dar escrita desnecessária ao restante do projeto.
6. Teste login, cadastro, edição, exclusão, visualização, download e permissões.
7. Na hospedagem, confirme HTTPS, cookies, regras de uploads e ausência de arquivos de desenvolvimento acessíveis ao público.

Mantenha as credenciais de cada ambiente fora do repositório público. Caso uma credencial seja exposta, substitua-a no provedor e atualize a configuração da aplicação. Remover um arquivo do último commit não elimina suas versões anteriores do histórico.

Não envie `.git`, dumps SQL, backups, logs ou arquivos de configuração temporários para uma pasta pública do servidor. O arquivo `painel/uploads/.htaccess` contém `Require all denied`; a eficácia precisa ser confirmada por requisição HTTP ao arquivo real. Os downloads devem continuar funcionando pelas rotas PHP.

### Erros e permissões locais

No ambiente XAMPP informado pelo autor, os erros PHP são gravados em `/opt/lampp/logs/php_error_log`. O log do Apache pode conter apenas inicialização e avisos do servidor. Um erro `move_uploaded_file(): Permission denied` indica que o usuário do servidor não conseguiu gravar no destino; não é resolvido mudando o botão de upload.

O usuário e grupo do Apache local foram identificados como `daemon`. Isso é específico daquele ambiente e não deve ser copiado automaticamente para a Hostinger. Não utilize permissão 777 como solução geral.

## 10. Manutenção e organização

### Diretórios principais

| Caminho | Conteúdo |
| --- | --- |
| Raiz do projeto | Páginas públicas, autenticação e rotas de visualização/download. |
| `assets/` | Estilos e recursos visuais do portal público. |
| `includes/` | Componentes compartilhados do portal e funções de entrega de arquivos e ícones. |
| `login/` | Interface de entrada dos operadores. |
| `painel/` | Páginas e ações administrativas. |
| `painel/includes/` | Sessão, autorização, CSRF, limite root e componentes do painel. |
| `painel/assets/` | Recursos visuais do painel. |
| `painel/uploads/` | Arquivos enviados; o acesso deve ocorrer pelas rotas controladas da aplicação. |

### Componentes compartilhados

`header.php` prepara a estrutura inicial da página e seus recursos visuais. `navbar.php` oferece a navegação superior. `sidebar.php` reúne os atalhos administrativos e marca a página ativa. `footer.php` encerra a estrutura e apresenta os elementos compartilhados do rodapé, conforme a área do sistema. Os arquivos de scripts concentram os recursos JavaScript previstos em cada layout.

Em `painel/includes/funcoes.php`, a função `listarDocumentos()` consulta o acervo para a listagem administrativa. A conexão utilizada por essa função vem de `painel/conexao.php`. Arquivos de configuração e funções compartilhadas devem ser incluídos com caminhos consistentes e `require_once`, evitando execução duplicada.

### Fluxo de uma alteração

1. O navegador solicita uma página administrativa.
2. A verificação de login confere a sessão e a conta no banco; ações root também exigem esse nível.
3. A página apresenta o formulário com o token CSRF.
4. O envio é recebido pelo arquivo responsável por salvar, atualizar ou excluir.
5. O servidor valida método, token, permissões e dados antes de executar a alteração.
6. O usuário retorna à listagem com uma mensagem de resultado.

### Cuidados ao modificar o código

- Preserve as verificações de acesso no início de cada página e ação protegida.
- Use consultas preparadas para valores recebidos do usuário e listas permitidas para opções de ordenação.
- Escape dados ao inseri-los em HTML, inclusive em atributos.
- Registre detalhes técnicos de falhas no log e apresente mensagens genéricas ao visitante.
- Valide os campos no servidor, mesmo quando o formulário já possui `required`, `maxlength` ou JavaScript.
- Ao modificar um ícone, campo ou filtro, confira todas as páginas que exibem ou recebem aquele dado.
- Antes de remover um arquivo físico, confira sua localização e possíveis referências no banco.

### Backups e recuperação

Mantenha cópias compatíveis do banco e de `painel/uploads/`, identificadas pela mesma data. Guardar apenas o banco preserva os registros, mas não o conteúdo dos documentos. Guardar apenas uploads perde nomes, categorias e relações do acervo.

Teste a restauração em ambiente separado: importe o banco, restaure os arquivos, configure a conexão e confira login, listagem, visualização e download. Restrinja o acesso aos backups, pois o banco contém dados de operadores e hashes de senha.

### Problemas comuns

| Sintoma | O que verificar |
| --- | --- |
| Serviço indisponível ao abrir o site | Conexão, disponibilidade do banco e log PHP. |
| Falha ao guardar arquivo | Permissão de escrita em uploads, espaço em disco e limites PHP. |
| Cadastro sem confirmação | Listagem e log antes de reenviar, evitando duplicações. |
| Ícones ausentes | Carregamento do Bootstrap Icons e classe salva na categoria. |
| Documento não abre | Existência do arquivo, caminho no banco, permissões e tipo detectado. |
| Aviso de constante já definida | Inclusão repetida do arquivo de conexão. |
| Sessão encerrada | Inatividade, desativação da conta ou alteração da versão da sessão. |

## 11. Roteiro de testes

Execute em cópia local com contas e documentos descartáveis. Registre data, versão do código, resultado esperado, resultado observado e evidência. Os cenários abaixo constituem um roteiro de validação; preencha os resultados somente após executá-los.

| Teste | Resultado esperado |
| --- | --- |
| Login correto e incorreto | Entrada apenas com credenciais válidas e conta ativa; mensagem sem detalhes internos na falha. |
| Seis tentativas de login na janela | A sexta é bloqueada, mesmo em outra sessão. |
| Acesso ao painel sem login | Redirecionamento ao login. |
| Administrador abre URL de ação root | Negação no servidor e nenhuma mudança no banco. |
| Token CSRF ausente ou adulterado | Requisição rejeitada e dados preservados. |
| Desativar conta com outra sessão aberta | Sessão antiga rejeitada na próxima requisição protegida. |
| PDF, JPG e PNG válidos | Cadastro, visualização e download funcionam. |
| Arquivo vazio, acima do limite ou tipo proibido | Rejeição sem publicação do arquivo. |
| Arquivo real por URL direta de uploads | Acesso negado; visualizar/download por ID continuam funcionando. |
| Alterar documento sem arquivo novo | Arquivo antigo mantido. |
| Substituir/excluir documento de teste | Registro consistente e limpeza do arquivo quando não houver outras referências. |
| Alterar ícone | Resultado igual no painel, início e página pública de categorias. |
| Pesquisa por `teste' OR '1'='1` | Tratamento como texto; não libera registros pela expressão nem revela SQL. |
| Nome de teste `<b>teste</b>` | Apresentado como texto nas listagens, sem ser interpretado como HTML. |
| Falha de consulta na listagem em ambiente isolado | Mensagem genérica, HTTP 503 e registro técnico apenas no log. |

Os dois testes de entrada acima são verificações pontuais; passar neles não comprova ausência de SQL injection ou XSS em todas as páginas. Testes com indisponibilidade do banco devem ocorrer apenas na cópia local, nunca interrompendo o portal publicado.

## 12. Continuidade do projeto

### Verificações antes de cada publicação

Verifique credenciais e histórico, valide o esquema real do banco, execute o roteiro de testes e confira a versão PHP, HTTPS e bloqueio de uploads. A configuração da hospedagem precisa ser conferida separadamente do código.

### Ajustes identificados na versão-base

Aplique e teste, caso ainda estejam pendentes:

- Inclusão única da conexão na listagem administrativa de documentos.
- Tratamento de falhas de consulta sem apresentar detalhes SQL ao visitante.
- Uso do ícone escolhido no painel também na página pública de categorias.

Esses ajustes não são considerados instalados apenas por estarem descritos nesta documentação.

### Evoluções futuras

Possibilidades para próximas versões: limite complementar por IP com tratamento de proxies, proteção contra enquadramento do painel em outros sites, validação UTF-8 uniforme nas categorias, confirmação de senha no cadastro de administradores, recuperação de senha, trilha de auditoria, rotina de conferência de arquivos órfãos e política de backup com teste de restauração.

A documentação técnica no GitHub serve à manutenção. A página Ajuda serve ao administrador que utiliza o painel. O PDF do TCC pode ser publicado como documento público somente após revisão e autorização da escola, sem senhas, dados pessoais indevidos ou configurações privadas.


### Orientações para quem continuar o projeto

Comece pela instalação local e execute o roteiro de testes antes de modificar as funcionalidades. Confira se a versão do banco corresponde ao código, registre as alterações no Git e atualize esta documentação junto com mudanças de comportamento. Combine com o responsável pela escola quais documentos podem ser publicados e quem terá acesso administrativo.

Novos recursos devem preservar o funcionamento da publicação, consulta e download. Recuperação de senha, auditoria e novos controles de acesso precisam de especificação e testes próprios antes de serem disponibilizados.
