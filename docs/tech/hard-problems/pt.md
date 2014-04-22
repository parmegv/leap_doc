@title = 'Problemas difíceis na comunicação segura'
@nav_title = 'Problemas difíceis'
@summary = "Como o LEAP aborda os problemas difíceis na comunicação segura"

## Os sete grandes

Se você pesquisar iniciativas interessantes para a criação de formas mais seguras de comunicação, verá que surge um padrão: aparentemente toda tentativa séria de construir um sistema para transmissão de mensagens seguras eventualmente se depara com a seguinte lista de sete problemas difíceis:

1. **Problema da autenticidade**: a validação de chaves públicas é muito difícil para ser gerenciada por usuários, mas sem isso não é possível obter confidencialidade.
2. **Problema dos metadados**: os protocolos existentes são vulneráveis à análise de metadados, mesmo que os metadados muitas vezes sejam mais sensíveis do que o conteúdo da comunicação.
3. **Problema da assincronicidade**: para estabelecer comunicação criptografada, atualmente é necessário escolher entre sigilo futuro (forward secrecy) e a habilidade de se comunicar de forma assíncrona.
4. **Problema do grupo**: na prática, pessoas trabalham em grupos, mas a criptografia de chave pública não.
5. **Problema dos recursos**: não existem protocolos abertos que permitam aos usuários compartilharem um recurso de forma segura.
6. **Problema da disponibilidade**: as pessoas querem alternar suavemente entre dispositivos e restaurar seus dados se perderem um dispositivo, mas isso é bem difícil de se fazer com segurança.
7. **Problema da atualização**: quase que universalmente, atualizações de software são feitas de maneiras que são convidativas a ataques e comprometimento de dispositivos.

Tais problemas parecem estar presentes independentemente da abordagem arquitetônica escolhida (autoridade centralizada, peer-to-peer distribuído ou servidores federados).

É possível ignorar muitos desses problemas se você não se importar especificamente com a usabilidade ou com o conjunto de funcionalidades com as quais os/as usuários/as se acostumaram nos métodos contemporâneos de comunicação online. Mas se você se importa com a usabilidade e recursos, então você terá que encontrar soluções para esses problemas.

## Nossas soluções

Em nosso trabalho, o LEAP tentou enfrentar diretamente esses sete problemas. Em alguns casos, chegamos a soluções sólidas. Noutros, estamos avançando com medidas paliativas temporárias e investigando soluções de longo prazo. Em dois casos não temos nenhum plano atual para lidar com os problemas.

### O problema da autenticidade

O problema:

> A validação de chaves públicas é muito difícil para ser gerenciada por usuários, mas sem isso não é possível obter confidencialidade.

Se a validação de chaves adequada é um pressuposto para uma comunicação segura, mas é muito difícil para a maioria dos usuários/as, que esperança temos?  Desenvolvemos um sistema federado único chamado [Nicknym](/nicknym) que descobre e valida automaticamente as chaves públicas, permitindo ao usuário tirar partido de criptografia de chave pública sem saber nada sobre chaves ou assinaturas.

O protocolo padrão que existe hoje para solucionar este problema chama-se [DANE](https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities). O DANE pode ser a melhor opção no longo prazo, mas atualmente é difícil de ser configurado, difícil de ser utilizado por clientes, vaza informações sobre associação para um observador da rede, e depende da confiança na zona raíz do DNS e nas zonas TLD.

### Problema dos metadados

O problema:

> Os protocolos existentes são vulneráveis à análise de metadados, mesmo que os metadados muitas vezes sejam mais sensíveis do que o conteúdo da comunicação.

Como medida de curto prazo, estamos integrando transporte criptografado oportunístico (TLS) para email e mensagens de chat ao serem retransmitidas entre servidores. Há dois aspectos importantes nisso:

* Servidores repetidores (relaying servers) precisam de uma maneira sólida para descobrir e validar as chaves uns dos outros. Para isso, estamos utilizando inicialmente DNSSEC/DANE.
* Um atacante não deve ser capaz de fazer o downgrade do transporte criptografado para texto não cifrado. Para isso, estamos modificando o software para assegurar que o transporte criptografado não possa sofrer downgrade.

Tal abordagem é potencialmente eficaz contra observadores externos na rede, mas não protege os metadados dos próprios provedores de serviços. Além disso, ela não protege, por si só, contra ataques mais avançados que envolvam análise de tráfego e de tempo.

No longo prazo, pretendemos adotar um dos vários esquemas distintos para roteamento seguro de metadados. Estes incluem:

* Pareamento automático de pseudônimos (auto-alias-pairs): cada uma das partes autonegocia pseudônimos para se comunicarem umas com as outras. Nos bastidores, o cliente utiliza de forma invisível esses pseudônimos para a comunicação subsequente. A vantagem é que isso é compatível com o roteamento existente. A desvantagem é que o servidor do usuário/a armazena uma lista de seus pseudônimos. Como uma melhoria, pode-se adicionar a possibilidade de usar um serviço de terceiros para manter o mapa dos pseudônimos.
* Cabeçalhos de roteamento do tipo "cebola" (onion-routing-headers): uma mensagem de um/a usuário/a para o/a usuário/a B é codificada de forma que as informações de roteamento do destinatário/a contenham apenas o nome do servidor usado por B. Quando o servidor de B recebe a mensagem, decodifica um cabeçalho adicional que contém o utilizador real "B". Como o uso de pseudônimos, isso não proporciona benefícios se os usuários estão no mesmo servidor. Como uma melhoria, a mensagem pode ser encaminhada por meio de servidores intermediários.
* Caixa de depósito de terceiros (third-party dropbox): para trocar mensagens, o/a usuário/a A e o/a usuário/a B negociam uma URL única de uma "caixa de depósito" (dropbox) para depositar mensagens, potencialmente usando um agente intermediário. Para enviar uma mensagem, o usuário A depositaria a mensagem na caixa. Para receber uma mensagem, o usuário B acessaria regularmente esta URL para ver se há novas mensagens.
* Misturador com assinaturas (mixmaster-with-signatures): as mensagens são enviadas através de um conjunto de repetirores anonimizadores do tipo mixmaster e ao final são entregues ao servidor do destinatário. O programa cliente do usuário apenas exibe a mensagem se ela for criptografada, tiver uma assinatura válida, e se o usuário tiver adicionado anteriormente o remetente a uma 'lista de permissões' (talvez gerada automaticamente a partir da lista de chaves públicas validadas).

Para uma boa discussão comparando redes misturadoras com roteamento cebola, veja a [postagem no blog de Tom Ritter](https://ritter.vg/blog-mix_and_onion_networks.html) sobre o tema.

### Problema da assincronicidade

O problema:

> Para estabelecer comunicação criptografada, atualmente é necessário escolher entre sigilo futuro (forward secrecy) e a habilidade de se comunicar de forma assíncrona.

Com o ritmo de crescimento do armazenamento digital e da criptanálise, o sigilo futuro é cada vez mais importante. Caso contrário, qualquer comunicação criptografada que você fizer hoje possivelmente se tornará uma comunicação em texto não cifrado num futuro próximo.

No caso do email e do bate-papo, existem o OpenPGP para email e o OTR para bate-papo: o primeiro fornece recursos assíncronos e o segundo fornece sigilo futuro, mas nenhum deles possuem ambas as habilidades. Precisamos tanto de uma melhor segurança para email quanto da capacidade de enviar e receber mensagens de bate-papo em modo offline.

No curto prazo, estamos empilhando transporte de email com sigilo futuro e relay de chat em cima de criptografia tradicional de objetos (OpenPGP). Esta abordagem é idêntica à nossa abordagem paliativa para o problema dos metadados, com o acréscimo de que os servidores repetidores precisam ter a capacidade de não apenas negociar transporte TLS mas também de negociar cifras que suportem sigilo futuro e que evitem uma precarização (downgrade) da cifra utilizada.

Esta abordagem é potencialmente eficaz contra os observadores externos na rede, mas não obtém sigilo futuro dos próprios prestadores de serviço.

No longo prazo, pretendemos trabalhar com outros grupos para criar novos padrões de protocolo de criptografia que podem ser tanto assíncronos quanto permitir o sigilo futuro:

  * [Extensões para sigilo futuro para o OpenPGP](http://tools.ietf.org/html/draft-brown-pgp-pfs-03).
  * [Handshake Diffie-Hellman triplo com curvas elípticas](https://whispersystems.org/blog/simplifying-otr-deniability/).

### Problema do grupo

O problema:

> Na prática, as pessoas trabalham em grupos, mas a criptografia de chave pública não.

Temos um monte de ideias, mas não temos ainda uma solução para corrigir este problema. Essencialmente, a questão é como usar primitivas de chaves públicas existentes para criar grupos criptográficos fortes, onde a adesão e as permissões são baseadas em chaves e em listas de controle de acesso mantidas no lado do servidor.

A maioria dos trabalhos interessantes nesta área tem sido feitos por empresas que trabalham com backup/sincronização/compartilhamento seguro de arquivos, como Wuala e Spideroak. Infelizmente, ainda não há quaisquer protocolos abertos bons ou pacotes de software livre que possam lidar com criptografia para grupos.

Neste momento, é provável que a melhor abordagem seja a abordagem simples: um protocolo no qual o cliente criptografa cada mensagem para cada destinatário individualmente, e que tenha algum mecanismo para verificação da transcrição de forma a garantir que todas as partes tenham recebido a mesma mensagem.

Existem alguns trabalhos em software livre com blocos construtivos interessantes que podem ser úteis na construção da criptografia para grupos. Por exemplo:

  * [Re-criptografia de proxy (proxy re-encryption)](https://en.wikipedia.org/wiki/Proxy_re-encryption): permite que o servidor cifre o conteúdo para novos beneficiários sem que tenha acesso ao texto não cifrado. O [gerenciador de lista de discussão SELS](http://sels.ncsa.illinois.edu/) usa OpenPGP para implementar um [sistema inteligente para o proxy de re-encriptação](http://spar.isi.jhu.edu/~mgreen/proxy.pdf).
  * [Assinaturas em anel (ring signatures)](https://en.wikipedia.org/wiki/Ring_signature): permite que qualquer membro do grupo assine, sem que se possa saber qual membro fez a assinatura.

### Problema dos recursos

O problema:

> Não existem protocolos abertos que permitam aos usuários compartilharem um recurso de forma segura.

Por exemplo, ao usar um bate-papo seguro ou rede social segura federada, você precisa de alguma forma de criar links para uma mídia externa, como uma imagem, vídeo ou arquivo, que tenha as mesmas garantias de segurança que a própria mensagem. A incorporação deste tipo de recurso nas mensagens em si é proibitivamente ineficiente.

Nós não temos uma proposta de como resolver este problema. Há um monte de grandes iniciativas que trabalham sob a bandeira da read-write-web, mas que não levam em conta a criptografia. De muitas maneiras, as soluções para o problema dos recursos são dependentes de soluções para o problema do grupo.

Tal como acontece com o problema do grupo, a maior parte do progresso nesta área tem sido por pessoas que trabalham em sincronização de arquivos criptografados (por exemplo, estratégias como a Revogação Preguiçosa -- Lazy Revocation -- e Regressão de Chave -- Key Regression).

### Problema da disponibilidade

O problema:

> As pessoas querem alternar suavemente entre dispositivos e restaurar seus dados se perderem um dispositivo, mas isso é bem difícil de se fazer com segurança.

Os/as usuários/as atuais exigem a capacidade de acessar seus dados em múltiplos dispositivos e de terem em mente que dados não serão perdidos para sempre se perderem um dispositivo. No mundo do software livre, só o Firefox abordou este problema adequadamente e de forma segura (com o Firefox Sync).

No LEAP, temos trabalhado para resolver o problema de disponibilidade com um sistema que chamamos de [Soledad](/soledad) (um acrônimo em inglês para "sincronização, entre dispositivos, de documentos criptografados localmente"). Soledad dá ao aplicativo cliente um banco de dados de documentos sincronizado, pesquisável e criptografado. Todos os dados são criptografados no lado do cliente, tanto quando ele é armazenado no dispositivo local quanto quando é sincronizado com a nuvem. Até onde sabemos, não há nada parecido com isso, seja no mundo do software livre ou comercial.

Soledad tenta resolver o problema genérico da disponibilidade de dados, mas outras iniciativas tentaram abordar o problema mais específico das chaves privadas e da descoberta de chaves públicas. Estas iniciativas incluem:

* [O protocolo proposto por Ben Laurie para armazenamento de segredos na nuvem](http://www.links.org/files/nigori/nigori-protocol-01.html).
* [Código para armazenamento de chaves na nuvem](https://github.com/mettle/nilcat), experimental e similar ao anterior.
* [Comentários de Phillip Hallam-Baker sobre questões similares](http://tools.ietf.org/html/draft-hallambaker-prismproof-key-00).

### O problema da atualização

O problema:

> Quase que universalmente, atualizações de software são feitas de maneiras que são convidativas a ataques e comprometimento de dispositivos.

O triste estado das atualizações de segurança é especialmente problemático porque os ataques de atualização já podem ser comprados prontos por regimes repressores. O problema de atualização de software é especialmente ruim em plataformas desktop. No caso aplicativos em HTML5 ou para dispositivos móveis, as vulnerabilidades não são tão terríveis, mas os problemas também são mais difíceis de corrigir.

Para resolver o problema da atualização, o LEAP está adotando um sistema de atualização exclusivo chamado Thandy do projeto Tor. Thandy é complexo para administrar, mas é muito eficaz na prevenção de ataques de atualização conhecidos.

Thandy, e o projeto relacionado [TUF](https://updateframework.com/), são projetados para dar conta das muitas [vulnerabilidades de segurança em sistemas de atualização de software](https://updateframework.com/projects/project/wiki/Docs/Security) existentes. Num exemplo, outros sistemas de atualização sofrem de uma incapacidade do cliente de confirmar que possuem a cópia mais recente, abrindo assim uma enorme vulnerabilidade onde o atacante simplesmente espera por uma atualização de segurança, evita que o upgrade ocorra e lança um ataque para a exploração da vulnerabilidade que deveria ter acabado de ser corrigida. Thandy/TUF fornecem um mecanismo único para a distribuição e verificação de atualizações de modo que nenhum dispositivo cliente irá instalar a atualização errada ou perder uma atualização sem saber.

Um problema relacionado com o problema da atualização é o problema do backdoor: como você sabe que uma atualização não tem um backdoor adicionado pelos próprios desenvolvedores do software? Provavelmente, a melhor abordagem é aquela tomada pelo [Gitian](https://gitian.org/), que fornece um "processo de construção determinística para permitir que vários construtores criem binários idênticos". Nós pretendemos adotar o Gitian no futuro.
