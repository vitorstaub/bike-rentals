# Explore Automated Machine Learning in Azure Machine Learning

Neste exercício, você utilizará o recurso de aprendizado de máquina automatizado no Azure Machine Learning para treinar e avaliar um modelo de aprendizado de máquina. Em seguida, você implantará e testará o modelo treinado.

## Criar um espaço de trabalho Azure Machine Learning

1. Faça login no portal Azure em [https://portal.azure.com](https://portal.azure.com) usando suas credenciais da Microsoft.
2. Selecione + Criar um recurso, procure por Machine Learning e crie um novo recurso Azure Machine Learning com as seguintes configurações:
   - Assinatura: Sua assinatura Azure.
   - Grupo de recursos: Crie ou selecione um grupo de recursos.
   - Nome: Insira um nome único para seu espaço de trabalho.
   - Região: Selecione a região geográfica mais próxima.
   - Conta de armazenamento: Anote a nova conta de armazenamento padrão que será criada para seu espaço de trabalho.
   - Key vault: Anote o novo cofre de chaves padrão que será criado para seu espaço de trabalho.
   - Application insights: Anote o novo recurso de insights de aplicativo padrão que será criado para seu espaço de trabalho.
   - Registro de contêiner: Nenhum (um será criado automaticamente na primeira vez que você implantar um modelo em um contêiner).
3. Selecione Revisar + criar e depois selecione Criar. Aguarde a criação de seu espaço de trabalho (isso pode levar alguns minutos) e, em seguida, vá para o recurso implantado.
4. Selecione Iniciar o studio (ou abra uma nova guia do navegador e acesse [https://ml.azure.com](https://ml.azure.com) e faça login no Azure Machine Learning studio usando sua conta da Microsoft). Feche quaisquer mensagens que sejam exibidas.

## Usar aprendizado de máquina automatizado para treinar um modelo

1. No Azure Machine Learning studio, visualize a página Automated ML (em Authoring).
2. Crie um novo trabalho de ML automatizado com as seguintes configurações:
   - Configurações básicas:
     - Nome do trabalho: mslearn-bike-automl
     - Novo nome do experimento: mslearn-bike-rental
     - Descrição: Aprendizado de máquina automatizado para previsão de aluguel de bicicletas
     - Tags: nenhum
   - Tipo de tarefa e dados:
     - Selecione o tipo de tarefa: Regressão
     - Selecione o conjunto de dados: Crie um novo conjunto de dados com as seguintes configurações:
       - Tipo de dados:
         - Nome: bike-rentals
         - Descrição: Dados históricos de aluguel de bicicletas
         - Tipo: Tabular
       - Fonte de dados:
         - Selecione De arquivos da web
         - URL da Web: [https://aka.ms/bike-rentals](https://aka.ms/bike-rentals)
         - Não selecione validação de dados ignorada
       - Configurações:
         - Formato do arquivo: Delimitado
         - Delimitador: Vírgula
         - Codificação: UTF-8
         - Cabeçalhos de coluna: Somente o primeiro arquivo tem cabeçalhos
         - Não pule linhas
       - Esquema:
         - Inclua todas as colunas exceto Path
         - Revise os tipos detectados automaticamente
     - Clique em Criar. Após a criação do conjunto de dados, selecione o conjunto de dados bike-rentals para continuar a enviar o trabalho de ML automatizado.

   - Configurações da tarefa:
     - Tipo de tarefa: Regressão
     - Conjunto de dados: bike-rentals
     - Coluna alvo: Aluguéis (inteiro)
     - Configurações adicionais de configuração:
       - Métrica primária: Erro médio quadrático normalizado
       - Explicar o melhor modelo: Não selecionado
       - Usar todos os modelos suportados: Não selecionado. Você restringirá o trabalho para tentar apenas alguns algoritmos específicos.
       - Modelos permitidos: Selecione apenas RandomForest e LightGBM - normalmente você gostaria de tentar o máximo possível, mas cada modelo adicionado aumenta o tempo necessário para executar o trabalho.
       - Limites: Expanda esta seção
         - Máximo de tentativas: 3
         - Máximo de tentativas simultâneas: 3
         - Máximo de nós: 3
         - Limiar de pontuação métrica: 0.085 (para que se um modelo atingir uma pontuação métrica de erro médio quadrático normalizado de 0.085 ou menos, o trabalho seja encerrado.)
         - Tempo limite: 15
         - Tempo limite de iteração: 15
         - Ativar término antecipado: Selecionado
       - Validação e teste:
         - Tipo de validação: Divisão de treinamento-validação
         - Porcentagem de dados de validação: 10
         - Conjunto de dados de teste: Nenhum
   - Computação:
     - Selecione o tipo de computação: Sem servidor
     - Tipo de máquina virtual: CPU
     - Nível da máquina virtual: Dedicado
     - Tamanho da máquina virtual: Standard_DS3_V2*
     - Número de instâncias: 1
     * Se sua assinatura restringir os tamanhos de VM disponíveis para você, escolha qualquer tamanho disponível.

Envie o trabalho de treinamento. Ele começa automaticamente.

## Revisar o melhor modelo

Quando o trabalho de aprendizado de máquina automatizado for concluído, você poderá revisar o melhor modelo treinado.

Na guia Visão geral do trabalho de aprendizado de máquina automatizado, observe o resumo do melhor modelo. Captura de tela do resumo do melhor modelo do trabalho de aprendizado de máquina automatizado com uma caixa ao redor do nome do algoritmo.

Observação: Você pode ver uma mensagem sob o status "Aviso: pontuação de saída especificada pelo usuário alcançada...". Esta é uma mensagem esperada. Por favor, continue para a próxima etapa.

Selecione o texto sob o nome do algoritmo para o melhor modelo para visualizar seus detalhes.

Selecione a guia Métricas e selecione os gráficos de resíduos e previsto_verdadeiro se eles ainda não estiverem selecionados.

Revise os gráficos que mostram o desempenho do modelo. O gráfico de resíduos mostra os resíduos (as diferenças entre os valores previstos e reais) como um histograma. O gráfico previsto_verdadeiro compara os valores previstos com os valores reais.

## Implantar e testar o modelo

Na guia Modelo para o melhor modelo treinado pelo seu trabalho de aprendizado de máquina automatizado, selecione Implantar e use a opção de serviço da Web para implantar o modelo com as seguintes configurações:

   - Nome: prever-aluguéis
   - Descrição: Prever aluguéis de bicicletas
   - Tipo de computação: Instância de contêiner Azure
   - Habilitar autenticação: Selecionado

Aguarde o início da implantação - isso pode levar alguns segundos. O status de Implantação para o endpoint prever-aluguéis será indicado na parte principal da página como Executando.

Aguarde o status de Implantação mudar para Concluído. Isso pode levar de 5 a 10 minutos.

## Testar o serviço implantado

Agora você pode testar seu serviço implantado.

No Azure Machine Learning studio, no menu à esquerda, selecione Pontos de extremidade e abra o endpoint de tempo real prever-aluguéis.

Na página de endpoint de tempo real prever-aluguéis, visualize a guia Teste.

Na painel de dados de entrada para testar o endpoint, substitua o JSON de modelo com os seguintes dados de entrada:

```
 {
   "Inputs": { 
     "data": [
       {
         "day": 1,
         "mnth": 1,   
         "year": 2022,
         "season": 2,
         "holiday": 0,
         "weekday": 1,
         "workingday": 1,
         "weathersit": 2, 
         "temp": 0.3, 
         "atemp": 0.3,
         "hum": 0.3,
         "windspeed": 0.3 
       }
     ]    
   },   
   "GlobalParameters": 1.0
 }
 ```

## Clique no botão Testar.

Revise os resultados do teste, que incluem um número previsto de aluguéis com base nas características de entrada - semelhante a isto:

```
 {
   "Results": [
     444.27799000000000
   ]
 }
```

## Referências

[https://microsoftlearning.github.io/01-machine-learning.html](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/01-machine-learning.html)

[https://microsoftlearning.github.io/02-content-safety.html](https://microsoftlearning.github.io/mslearn-ai-fundamentals/Instructions/Labs/02-content-safety.html)