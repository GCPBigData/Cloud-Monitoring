# Cloud-Monitoring

O Cloud Monitoring coleta métricas, eventos e metadados do Google Cloud e da Amazon Web Services (AWS), bem como sondagem de tempos de atividade hospedados e instrumentação de aplicativos. Com o serviço BindPlane, também é possível coletar esses dados de mais de 150 componentes de aplicativos em comum, sistemas no local e de nuvem híbrida. O pacote de operações do Google Cloud processa esses dados e gera insights por meio de painéis, gráficos e alertas. O BindPlane está incluído no seu projeto do Google Cloud sem custo adicional.

Para coletar dados de métricas das instâncias do Compute Engine, crie uma política de agente que instale e mantenha automaticamente as operações do Google Cloud conjunto de agentes na sua frota de VMs.

O Cloud Monitoring é uma grande coleção de ferramentas que permitem responder a perguntas importantes:

"Meu serviço é íntegro?"
"Qual é a carga no meu serviço?"
"Meu site está funcionando e respondendo corretamente?"
"Meu serviço tem um bom desempenho?"
O Cloud Monitoring mede os principais aspectos dos seus serviços, oferece a capacidade de representar as medições em gráficos e notifica você quando elas não têm valores aceitáveis. Este documento fornece uma breve visão geral desses recursos.

Técnicas de monitoramento
O Cloud Monitoring oferece quatro tipos de monitoramento:

Com o monitoramento de caixa preta, é possível testar o serviço da mesma forma que um usuário faria: solicitando uma página da Web, conectando-se a uma porta TCP ou fazendo uma chamada de API REST. Esse tipo de monitoramento não fornece informações sobre o funcionamento interno do serviço. O serviço é tratado como uma entidade opaca. O Cloud Monitoring fornece esse tipo de monitoramento com verificações de tempo de atividade. Para mais informações, consulte Como monitorar um serviço com verificações de tempo de atividade.

O monitoramento de caixa branca permite que você monitore aspectos do seu serviço que são importantes para você. Você pode instrumentar seu serviço para gravar dados com carimbo de data/hora usando uma biblioteca como o OpenCensus ou gravar dados de série temporal personalizados usando a API Cloud Monitoring. Para mais informações, consulte Como usar métricas personalizadas.

O monitoramento de caixa cinza coleta informações sobre o estado do ambiente em que seus serviços estão sendo executados. Esse tipo de monitoramento é fornecido por uma combinação de produtos do Google Cloud e parceiros terceirizados do Cloud Monitoring, como a Blue Medora. Exemplo:

Os serviços do Google Cloud geram métricas que fornecem informações sobre como o serviço está operando. Por exemplo, o Compute Engine informa o uso da CPU e a utilização da CPU de cada instância de VM, ela também informa a contagem de bytes e pacotes descartados pelo firewall. Para uma lista completa, consulte métricas do Google Cloud.
O agente do Cloud Monitoring reúne métricas do sistema e do aplicativo. Quando instalado nas instâncias de VM do Compute Engine, este agente coleta métricas de processo, disco, CPU e rede. Quando instalado no Linux, o agente também pode ser configurado para coletar métricas de plug-ins de terceiros.
Plug-ins de terceiros fornecem dados no nível de serviço nas suas VMs do Linux. Essas informações podem incluir métricas sobre os servidores da Web Apache ou Nginx ou métricas sobre os bancos de dados MongoDB ou PostgresSQL.
As métricas com base em registros são coletadas do conteúdo dos registros gravados no Cloud Logging. As métricas com base em registros predefinidas incluem, por exemplo, erros que o serviço detecta ou o número total de entradas de registro recebidas. Também é possível definir métricas com base em registros personalizadas. Por exemplo, é possível contar o número de entradas de registro que correspondem a uma determinada consulta ou acompanhar determinados valores nas entradas de registro correspondentes.

Recursos monitorados, descritores de métrica e séries temporais
Nesta seção, apresentamos a terminologia usada pelo Cloud Monitoring. Para uma discussão mais detalhada dos conceitos apresentados nesta seção, consulte Estrutura de séries temporais.

Recursos monitorados
Um recurso monitorado é o componente de hardware ou software que está sendo monitorado. Exemplos de recursos monitorados incluem discos e instâncias do Compute Engine, além de aplicativos e instâncias do App Engine. Há cerca de 100 tipos de recursos monitorados disponíveis. Para ver a lista atual, consulte Lista de recursos monitorados.

Cada tipo de recurso monitorado é descrito formalmente em uma estrutura de dados chamada MonitoredResourceDescriptor. Por exemplo, veja o descritor de recurso monitorado para o recurso gce_instance:


{
  "type": "gce_instance",
  "displayName": "G​C​E VM Instance",
  "description": "A virtual machine instance hosted in Compute Engine (G​C​E).",
  "name": "projects/PROJECT_ID/monitoredResourceDescriptors/gce_instance"
  "labels": [
    {
      "key": "project_id",
      "description": "The identifier of the Google Cloud project associated with this resource, such as \"my-project\"."
    },
    {
      "key": "instance_id",
      "description": "The numeric VM instance identifier assigned by Compute Engine."
    },
    {
      "key": "zone",
      "description": "The Compute Engine zone in which the VM is running."
    }
  ],
}
Quando um serviço grava dados no Cloud Monitoring, os dados a serem gravados sempre se referem a um recurso monitorado. Se você visualizar esses dados, poderá usar esses rótulos, representados como pares de chave-valor, para identificar o que gerou os dados. Por exemplo, se os dados se referirem ao recurso monitorado gce_instance, será possível identificar a instância de VM específica visualizando o valor do rótulo instance_id.

No Cloud Monitoring, ao criar gráficos ou políticas de alertas, é possível filtrar e agrupar os dados com base nos valores dos rótulos em uma série temporal. Por exemplo, se você tiver um projeto do Google Cloud com várias instâncias de VM do Compute Engine, poderá criar um gráfico que exibe a série temporal da utilização da CPU de cada instância. Se você adicionar um filtro para instance_id, poderá exibir apenas a utilização da CPU de uma instância de VM específica.

Descritores de métrica
Todos os dados enviados ao Cloud Monitoring são descritos por um descritor de métrica. Um descritor de métrica é uma definição que descreve os atributos dos dados. O Cloud Monitoring tem aproximadamente 1.500 descritores de métricas integrados. Consulte a Lista de métricas para ver detalhes.

Veja a seguir um exemplo de descritor de métrica:


    Metric type: agent.googleapis.com/disk/percent_used
    Display name: Disk utilization
    Metric kind: GAUGE
    Value type: DOUBLE
    Units: % (this symbol indicates a percentage which is a value between 0.0 and 100.0)
    Labels: device, state
    Monitored resource: gce_instance (this value refers to a Compute Engine VM instance)
No restante desta seção, descrevemos alguns dos principais atributos de um descritor de métrica. Para ver uma descrição completa, consulte MetricDescriptor:

Tipo de métrica

O Tipo de métrica é semelhante a um URL. Para os serviços do Google Cloud e para determinadas integrações de terceiros, a primeira parte do tipo de métrica identifica a origem da série temporal e o restante descreve o que está sendo monitorado. No descritor de métrica de exemplo, agent.googleapis.com identifica a origem como o agente do Cloud Monitoring e disk/percent_used indica que a porcentagem usada é medida. Para métricas personalizadas definidas pelo usuário, o tipo de métrica é prefixado por external.googleapis.com ou custom.googleapis.com.

Os tipos de métrica são globalmente exclusivos.

Como o tipo de métrica é globalmente exclusivo, os termos descritor de métrica e tipo de métrica geralmente são trocados. No Console do Google Cloud, o termo métrica é frequentemente usado no lugar de tipo de métrica.

Nome de exibição

O Nome de exibição é um nome curto e descritivo para o descritor de métrica. No exemplo, o nome de exibição é "Utilização do disco". O nome de exibição, que pode não ser exclusivo, é usado no Console do Google Cloud para simplificar a exibição de dados.

Classe da métrica

O tipo de métrica descreve a relação entre valores medidos adjacentes em uma série temporal:

As métricas GAUGE armazenam o valor daquilo que está sendo medido em um determinado momento. Uma analogia é o velocímetro do seu carro, que registra sua velocidade atual.

As métricas CUMULATIVE armazenam o valor acumulado daquilo que está sendo medido em um determinado momento. Uma analogia é o odômetro do seu carro, que registra a distância total que você percorreu.

As métricas DELTA armazenam a alteração no valor daquilo que está sendo medido durante um período especificado. Uma analogia é o odômetro de viagem, redefinido todos os dias, que mede a distância total percorrida durante esse dia ou desde a última redefinição. Outro exemplo é um resumo de ações que informa quanto dinheiro você ganhou ou perdeu no mercado hoje.

Para ver mais informações, consulte MetricKind.

Tipo de valor

O Tipo de valor descreve o tipo de dados da medição. As medições numéricas são INT64 ou DOUBLE. As métricas também podem ter valores do tipo BOOL, STRING ou DISTRIBUTION. Todos os pontos de dados de uma série temporal têm o mesmo tipo de valor.

Para mais informações sobre os tipos de valor, consulte ValueType.

Unidade de métrica

A Unidade de métrica descreve a unidade ou medida em que um ponto de dados é registrado.

Por exemplo, By é a notação padrão para indicar "bytes" e kBy é kilobytes, ou "milhares de bytes". Para registrar que 1126 bytes foram gravados, se a unidade for kBy, o valor será 1.126. As unidades de métrica disponíveis também incluem aquelas adequadas para informações digitais. Por exemplo, KiBy é "1024 bytes". Para registrar que 1126 bytes foram gravados, se a unidade for KiBy, o valor será 1.099.

Para mais informações sobre unidades de métricas, consulte Unidades.

Rótulos

Alguns descritores de métrica especificam rótulos para aumentar os rótulos definidos no recurso monitorado. É possível filtrar e agrupar os dados pelo valor do rótulo quando você cria gráficos ou políticas de alerta.

O descritor de métrica de exemplo inclui os rótulos de device e state. O rótulo device indica o identificador do disco e o rótulo state identifica se a série temporal contém valores para espaço livre, usado ou reservado em disco. Se você visualizar dados que mostram o uso de disco de uma instância de VM do Compute Engine, estará vendo dados para o tipo de métrica agent.googleapis.com/disk/percent_used gravados em uma instância de VM do Compute Engine. Os dados contêm cinco rótulos: os três especificados no descritor de recurso monitorado e os dois definidos no descritor de métrica. Usando filtros, é possível, por exemplo, visualizar apenas o espaço livre em disco ou apenas o espaço livre em uma VM específica.

Série temporal
Uma série temporal é uma coleção de medições e metadados sobre essas medidas. Veja a seguir uma parte de uma série temporal:


{
  "timeSeries": [
    {
      "metric": {
        "labels": {
          "device": "sda1",
          "state": "free"
        },
        "type": "agent.googleapis.com/disk/percent_used"
      },
      "resource": {
        "type": "gce_instance",
        "labels": {
          "instance_id": "2708613220420473591",
          "zone": "us-east1-b",
          "project_id": "sampleproject"
        }
      },
      "metricKind": "GAUGE",
      "valueType": "DOUBLE",
      "points": [
        {
          "interval": {
            "startTime": "2020-07-27T20:20:21.597143Z",
            "endTime": "2020-07-27T20:20:21.597143Z"
          },
          "value": {
            "doubleValue": 0.473005
          }
        },
        {
          "interval": {
            "startTime": "2020-07-27T20:19:21.597239Z",
            "endTime": "2020-07-27T20:19:21.597239Z"
          },
          "value": {
            "doubleValue": 0.473025
          }
        },
      ],
    },
Cada série temporal tem uma coleção única de valores de rótulo. Como ilustramos neste exemplo, a série temporal contém rótulos de métricas e de recursos. Os valores dos rótulos mostram que a série temporal é o espaço livre do disco "sda1", que faz parte da instância de VM com ID 2708613220420473591. Há uma série temporal diferente para o espaço livre do disco e outra para o espaço reservado do disco. O exemplo contém duas medições, em que cada uma delas é um ponto representado por um intervalo de tempo e um valor.

Como visualizar dados de série temporal
Para visualizar dados de série temporal, use o Metrics Explorer ou visualize um gráfico em um painel.

Metrics Explorer
O Metrics Explorer fornece uma interface orientada por menus em que você seleciona o tipo de recurso monitorado e o tipo de métrica para os quais você quer visualizar dados de série temporal. Depois de fazer essas seleções, é possível aplicar filtros para exibir apenas séries temporais específicas. Para permitir o gerenciamento de configurações complexas, o Metrics Explorer fornece um conjunto de opções de agregação. Para mais informações sobre essas opções, consulte Filtragem e agregação.

Por exemplo, a captura de tela a seguir ilustra a utilização média de discos livres, usados e reservados para os discos de todas as instâncias de VM localizadas na zona "us-central1-a":

Metrics Explorer exibindo a utilização do disco.

Se você quiser examinar as tendências, configure o Metrics Explorer para comparar os dados de série temporal atuais com os dados anteriores.

Para realizar uma análise comparativa de dados diferentes, crie gráficos que exibem dados de série temporal de vários descritores de métrica. Por exemplo, você pode exibir dados de "Tráfego", "Latência" e "Vendas" no mesmo gráfico.

Para mais informações, consulte Metrics Explorer.

Gráficos e painéis
Para exibir informações sobre um conjunto de recursos, use os painéis. O Cloud Monitoring é compatível com dois tipos diferentes de painéis:

Os painéis pré-configurados são criados automaticamente pelo Cloud Monitoring quando um recurso está em uso pelo serviço. Por exemplo, se o serviço for criado usando um servidor da Web Apache no Google Cloud, os painéis serão criados automaticamente para o servidor da Web, para cada um dos discos do Compute Engine, para firewalls e instâncias de VM do Compute Engine. Se os discos forem armazenados em backup por snapshots, painéis adicionais serão criados automaticamente.

Painéis pré-configurados são projetados para exibir as informações mais visualizadas. Por exemplo, o painel de uma instância de VM do Compute Engine inclui informações sobre a zona, endereços IP públicos e privados e gráficos que exibem o uso da CPU e outros dados interessantes.

Com os Painéis personalizados, você cria uma coleção de gráficos que quer visualizar. Para adicionar um gráfico a um painel, você pode usar o recurso "adicionar gráfico" em um painel ou criar um gráfico com o Metrics Explorer e salvá-lo no painel. Se você usar o Console do Google Cloud para criar um painel, os gráficos serão exibidos em um padrão de grade. No entanto, você pode criar configurações mais complexas usando a API Dashboards.

Para mais informações sobre gráficos e painéis, consulte Como usar painéis e gráficos.

Como monitorar dados de série temporal
Para receber uma notificação quando a série temporal atender a determinadas condições, crie uma política de alertas. Você pode criar políticas de alerta simples e complexas, por exemplo:

"Notificar se uma verificação de tempo de atividade falhar por mais de três minutos em qualquer local."

"Notificar a equipe da chamada se o 90º percentil das respostas HTTP 200 de três ou mais servidores da Web em dois locais distintos do Google Cloud exceder uma latência de resposta de 100 ms, desde que haja. menos de 15 QPS no servidor".

Para gerenciar e visualizar suas políticas, o Cloud Monitoring fornece um painel de alerta.

Esta seção fornece uma breve visão geral dos alertas. Para mais informações, consulte Introdução aos alertas.

Componentes da política de alertas
No Cloud Monitoring, uma política de alertas consiste em quatro componentes:

Um nome exibido no painel de alerta e incluído nas notificações enviadas.
Uma lista de canais de notificação que especificam quem será notificado. O Monitoring é compatível com canais de notificação comuns. É possível configurar uma notificação para ser enviada por e-mail, para um dispositivo móvel ou um serviço, como o PagerDuty ou o Slack. Para uma lista completa, consulte Opções de notificação.

Documentação personalizada a ser incluída na notificação. Por exemplo, configure esse conteúdo para descrever quais ações devem ser realizadas por um operador humano. Este campo suporta o uso de variáveis parametrizadas. Para mais informações, consulte Variáveis em modelos de documentação.

Uma ou mais condições que a política de alertas avalia. Cada condição especifica um recurso monitorado, um tipo de métrica e quando essa condição é atendida. Por exemplo, uma condição pode monitorar a utilização do disco de uma instância de VM e ser atendida se o espaço livre for menor que 10% por pelo menos 5 minutos.

Quando as condições de uma política de alertas são atendidas, um incidente é gerado e as notificações são emitidas. Quando as condições não são mais atendidas, o incidente é resolvido automaticamente e outra notificação é enviada aos canais de notificação especificados.

Exemplo: política de alertas para espaço livre em disco
Suponha que você queira ser notificado se o espaço em disco livre estiver abaixo de 35% para qualquer disco chamado tmpfs nas instâncias de VM na zona "us-central1-a".

Você decide criar uma política de alertas. Para isso, deve fazer o seguinte:

No Console do Google Cloud, acesse o Cloud Monitoring e clique em Alerta.

Clique em Criar política para criar uma nova política de alertas. Na caixa de diálogo exibida, insira um nome e selecione Adicionar condição.

Na caixa de diálogo de condição, selecione o recurso monitorado, o tipo de métrica e aplique os filtros:

Selecione o tipo de recurso.

Para esse alerta, é preciso tomar as medidas a seguir:

Em Tipo de recurso, selecione VM instance.
Em Métrica, selecione Disk Utilization
Adicione filtros para a zona, o estado e o disco:

zone = "us-central1-a"
state = "free"
disk = "tmpfs"
Com essas opções, o gráfico interativo exibe o espaço livre em cada disco "tmpfs" na zona "us-central1-a":

Política de alertas exibindo a utilização do disco.

A captura de tela mostra que, em um projeto, há dois discos "tmpfs" na zona "us-central1-a".

Você quer que a condição seja atendida quando "o espaço livre em disco para qualquer disco estiver abaixo de 35%". Insira essas informações no painel Configuração:

Condição que exibe a configuração.

Defina a seção Condition triggers if como Any time series violates.

Você escolheu esse valor porque quer ser notificado se alguma das séries temporais tiver um valor abaixo de 35%. Há outras opções disponíveis. Por exemplo, é possível definir esse campo para todas as séries temporais, um número específico de séries temporais ou uma porcentagem delas.

Defina Condition como is below e o Threshold como 35%.

Você escolheu essas configurações porque quer comparar o valor da série temporal com 35% e quer ser notificado se o valor estiver abaixo desse número. Outras opções incluem estar acima, ausente ou pela velocidade com que o valor muda.

Defina o campo For como most recent value.

Você escolheu o valor mais recente porque quer ser notificado imediatamente quando o valor do espaço livre em disco for menor que 35%. O campo For define uma duração. Se esse campo estiver definido como cinco minutos, o espaço livre em disco precisará ser inferior a 35% por cinco minutos antes que a notificação ocorra. Nesse caso, o valor padrão do campo de duração foi definido como um minuto, o que é longo o suficiente para garantir que uma única medição não faça com que um incidente seja criado.

Para concluir a política, salve a condição, adicione os canais de notificação e adicione a documentação.

Neste exemplo, quando a condição é atendida, um incidente é criado e as notificações são enviadas.

Como monitorar um serviço com verificações de tempo de atividade
O Cloud Monitoring oferece monitoramento de caixa preta com verificações de tempo de atividade. Se você configurar uma verificação de tempo de atividade para um serviço, a capacidade de resposta do serviço será periodicamente testada por servidores localizados em pelo menos três locais diferentes.

A verificação de tempo de atividade registra o sucesso ou a falha da resposta e a latência da resposta como séries temporais. O Cloud Monitoring cria um painel para cada verificação de tempo de atividade no seu projeto do Google Cloud. Em um painel de verificações de tempo de atividade, é possível visualizar a latência das respostas, o histórico de respostas e informações detalhadas sobre a verificação. Também é possível visualizar esses dados com a criação de um gráfico com o Metrics Explorer ou adicionando um gráfico a um painel personalizado. Para as configurações de gráfico, consulte Como criar um gráfico de latência de tempo de atividade.

Também é possível configurar a verificação de tempo de atividade para ser associada a uma política de alertas. Com essa configuração, a política de alertas notifica você se a verificação de tempo de atividade falhar. Para mais informações, consulte Verificações de tempo de atividade.

O Cloud Monitoring fornece um painel de tempo de atividade que exibe um resumo das verificações de tempo de atividade. É possível filtrar a exibição, conforme mostra a captura de tela a seguir, e você pode usar os links incorporados para visualizar os detalhes de uma verificação de tempo de atividade específica.

Visão geral das verificações de tempo de atividade com filtros de exemplo.

Como monitorar grupos
Um grupo do Cloud Monitoring é uma coleção de recursos do Google Cloud ou da AWS que tem atributos específicos definidos por você. Exemplos de grupos incluem "todas as instâncias do Compute Engine em que o nome começa com uma string específica", "todos os recursos com determinadas tags" e "todos os recursos de computação da AWS na região A ou B". À medida que você adiciona e remove recursos, a associação no grupo é alterada automaticamente.

Se você configurar uma verificação de tempo de atividade para monitorar um grupo, ela verificará todos os membros do grupo quando a sondagem for emitida.

Se os recursos forem adicionados ou removidos, a verificação de tempo de atividade ajustará automaticamente o que for testado. No painel dessa verificação, é possível ver a latência e a série temporal de verificação aprovada para cada membro do grupo. Também é possível configurar esses tipos de verificações de tempo de atividade para associá-los a uma política de alertas.

Ao criar um gráfico ou uma política de alertas, é possível filtrar as séries temporais por nome do grupo do Cloud Monitoring e agrupar séries temporais pelo nome do grupo.

Para mais informações, consulte Como usar grupos de recursos.

Monitoramento de espaços de trabalho
O Cloud Monitoring usa espaços de trabalho como um mecanismo para permitir que você visualize e gerencie dados de séries temporais armazenados em vários projetos do Google Cloud em um único local. O espaço de trabalho armazena gráficos, painéis, verificações de tempo de atividade e outras ações de configuração que você realiza.

Os espaços de trabalho foram projetados para ser transparentes para a maioria dos usuários. No cenário mais simples e comum, quando você acessa o Monitoring no Console do Google Cloud pela primeira vez, um espaço de trabalho é criado automaticamente para seu projeto. Nesse caso mais simples, o espaço de trabalho monitora um único projeto do Google Cloud.

Depois de criar um espaço de trabalho, na página Configuração dele, é possível adicionar outros projetos do Google Cloud. Ao adicionar um projeto, você ativa o Cloud Monitoring para ler os dados de série temporal armazenados nele.

Para uma visão geral conceitual, consulte Espaços de trabalho.

Temas relacionados
Para monitorar os serviços da maneira como o Google gerencia os próprios serviços, consulte Conceitos no monitoramento de serviço. Com o monitoramento de serviço, você define os objetivos de desempenho do serviço, como medir o desempenho e um erro de orçamento. Crie políticas de alertas para notificar você quando o erro de orçamento estiver sendo consumido mais rápido que o desejado.

Se você estiver usando o Google Kubernetes Engine, consulte Visão geral do pacote de operações do Google Cloud para GKE, que descreve como observar o GKE usando o Cloud Monitoring e o Cloud Logging.

Primeiros passos com o Monitoring
Esta página fornece uma breve visão geral dos principais componentes do Cloud Monitoring.

Para explorar os recursos do Cloud Monitoring, consulte o Guia de início rápido do Cloud Monitoring para o Compute Engine. O guia de início rápido orienta você na criação de uma verificação de tempo de atividade, uma política de alertas e um painel personalizado usando o console do Cloud Monitoring. O console do Cloud Monitoring fornece uma interface orientada a menus e, em algumas situações, é compatível com a linguagem de consulta do Monitoring. MQL é uma interface expressiva baseada em texto para consultar dados de série temporal.

Você também pode usar a interface programática do Cloud Monitoring, a API Cloud Monitoring, para criar e gerenciar gráficos e painéis, verificações de tempo de atividade, políticas de alertas e grupos. A API é compatível com a linguagem tradicional baseada em filtros e MQL. Para mais informações, consulte Como criar um gráfico com a linguagem de consulta do Monitoring e Como criar um painel pela API.

Para mais informações sobre o Monitoring, consulte os recursos abaixo:

Como criar e gerenciar verificações de tempo de atividade
Como criar e gerenciar políticas de alerta
Como criar e gerenciar painéis e gráficos
Como criar gráficos usando o Metrics Explorer
Como visualizar e modificar suas configurações do espaço de trabalho
Como gerenciar incidentes e eventos
Para informações sobre preços, cotas e limites, consulte Recursos.

Para ver uma lista de recursos monitorados, consulte Lista de recursos monitorados.

Para uma lista de métricas compatíveis, consulte a Lista de métricas.

Para mais informações sobre a API Cloud Monitoring, consulte:

Como ativar a API
Usar o API Explorer
APIs e referência