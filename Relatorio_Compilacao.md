# Relatório de Compilação e Automação - DayZ Loader

Este documento detalha como compilar o seu loader. **Novidade:** Agora o projeto conta com compilação automática via GitHub! Basta subir os arquivos para o seu repositório e o GitHub gera o `.exe` para você.

## Requisitos do Sistema

Para realizar a compilação do projeto com sucesso, é imprescindível ter o ambiente de desenvolvimento corretamente configurado. O projeto foi estruturado para ser compilado utilizando o **Visual Studio 2022** (sendo compatível também com a versão 2019). Durante a instalação do Visual Studio, certifique-se de selecionar a carga de trabalho voltada para o **Desenvolvimento para Desktop com C++**, garantindo que o componente do SDK do Windows 10 ou 11 seja incluído na instalação.

Além do ambiente da Microsoft, o projeto faz uso de bibliotecas gráficas legadas. Portanto, é obrigatória a instalação do **DirectX SDK (June 2010)**. Este SDK fornece as bibliotecas essenciais `d3d9.lib` e `d3dx9.lib` requeridas pelo projeto. Caso enfrente o erro "S1023" durante a instalação do DirectX SDK em sistemas modernos, a solução padrão consiste em desinstalar temporariamente os pacotes do "Microsoft Visual C++ 2010 Redistributable" presentes no sistema, realizar a instalação do SDK e, em seguida, os pacotes serão restaurados automaticamente ou poderão ser reinstalados.

## Estrutura do Projeto

O código-fonte foi limpo e organizado para facilitar a manutenção. O arquivo principal da solução do Visual Studio está localizado no caminho `examples\example_win32_directx9\DayZLoader.vcxproj`.

O arquivo mais importante do projeto é o `main.cpp`. Nele está concentrada toda a lógica de funcionamento do loader, incluindo a renderização da interface gráfica utilizando a biblioteca ImGui, as rotinas de requisição HTTP para o download da DLL diretamente do GitHub, a geração de nomes aleatórios para o arquivo e, por fim, a lógica de injeção da DLL na memória do processo alvo utilizando a técnica de `LoadLibrary` remoto. Os arquivos `options.cpp` e `options.hpp` foram mantidos e simplificados para armazenar variáveis globais de configuração. Os arquivos base do framework ImGui encontram-se distribuídos na raiz do projeto e na pasta de backends.

## Opção 1: Compilação Automática (GitHub Actions) - RECOMENDADO

Esta é a forma mais fácil de obter o seu `.exe` sem precisar instalar nada no seu PC:

1. Crie um novo repositório no seu GitHub.
2. Suba todos os arquivos desta pasta para o repositório.
3. No seu GitHub, clique na aba **"Actions"** no topo da página.
4. Você verá um processo chamado "Build DayZ Loader". Quando ele terminar (ficar verde), clique nele.
5. Role até o final da página e, em **"Artifacts"**, você verá o arquivo `DayZ-Loader-Exe`. Clique para baixar.
6. Pronto! O `.exe` compilado estará dentro do arquivo baixado.

## Opção 2: Compilação Manual (Seu PC)

Para gerar o executável final do seu loader, o primeiro passo é abrir o arquivo de projeto `DayZLoader.vcxproj` com o Visual Studio. 

Em seguida, é importante verificar as configurações de diretório do DirectX. Por padrão, o projeto está configurado para utilizar a variável de ambiente `$(DXSDK_DIR)`. Caso o compilador apresente erros informando que não encontrou os arquivos de cabeçalho do DirectX, será necessário informar os caminhos manualmente. Para isso, acesse as propriedades do projeto clicando com o botão direito sobre ele no *Solution Explorer*. Na seção "C/C++" e subseção "Geral", adicione o caminho da pasta `Include` do DirectX SDK aos diretórios de inclusão adicionais. De forma análoga, na seção "Vinculador" (Linker) e subseção "Geral", adicione o caminho da pasta `Lib\x86` (ou x64) aos diretórios de biblioteca adicionais.

Com os caminhos configurados, selecione a plataforma alvo na barra superior do Visual Studio. É altamente recomendado compilar o projeto na configuração **Release**. Quanto à arquitetura, selecione **x64** caso a sua DLL e o jogo (DayZ) operem em 64 bits, ou **x86** caso contrário. 

Por fim, acesse o menu "Compilar" (Build) e selecione "Compilar Solução" (Build Solution). O Visual Studio iniciará o processo de compilação. Ao término, uma mensagem de sucesso será exibida na janela de saída (Output). O arquivo executável gerado estará disponível na pasta `bin\Release_x64\` (ou x86), localizada dentro do diretório do projeto.

## Instruções de Uso

Com o executável compilado em mãos, a utilização do loader ocorre em duas etapas principais. 

Primeiramente, é necessário realizar o download da DLL. Abra o executável gerado e navegue até a aba "Configurações" na interface do loader. Insira a URL em formato RAW do seu repositório no GitHub que aponta diretamente para o arquivo da DLL. Em seguida, utilize o botão de busca para selecionar em qual diretório do seu computador o arquivo será salvo. O loader se encarregará de gerar um nome de arquivo completamente aleatório para evitar detecções por assinatura de nome. Clique no botão de download e aguarde a confirmação de que o arquivo foi salvo com sucesso.

A segunda etapa consiste na injeção propriamente dita. Navegue para a aba "Loader". O campo de nome do processo já estará preenchido com `DayZ.exe` por padrão. Inicie o jogo DayZ e, com ele aberto, clique no botão para atualizar a lista de processos no loader. O identificador do processo (PID) do jogo deverá ser detectado e exibido na cor verde. Com a DLL baixada e o processo selecionado, clique no botão principal "LOAD / INJECT" para que a DLL seja injetada na memória do jogo.

**Atenção:** Para que as funções de leitura e escrita de memória (`OpenProcess`, `CreateRemoteThread`) funcionem corretamente, é estritamente necessário executar o Loader com privilégios de Administrador no Windows.
