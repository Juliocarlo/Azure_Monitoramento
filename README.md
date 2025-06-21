# Azure_Monitoramento
Informações sobre o monitoramento no Azure 

O que é o Azure Monitor?

O Azure Monitor é a plataforma de observabilidade nativa da Microsoft para monitorar a integridade, 
desempenho e disponibilidade de recursos no Azure, ambientes híbridos e multicloud. 
Ele coleta, analisa e visualiza métricas, logs, telemetria de aplicativos e eventos de rede.

Componentes Principais
| Componente           | Função                                                       | 

| Métricas             | Dados numéricos em tempo real (CPU, memória, latência etc.)  | 

| Logs (Log Analytics) | Dados estruturados e não estruturados para análise detalhada | 

| Application Insights | Monitoramento de desempenho de aplicações (APM)              | 

| Alertas              | Notificações automáticas com base em condições definidas     | 

| Workbooks            | Dashboards interativos e personalizáveis                     | 

| Autoscale            | Escalonamento automático com base em métricas                | 

| Network Insights     | Diagnóstico de conectividade e desempenho de rede            | 

| Change Analysis      | Histórico de alterações em recursos                          | 


Novidades e Atualizações Recentes
- Unified Data Collection Rules (DCR): coleta de dados centralizada e mais eficiente.
- Managed Prometheus + Grafana: suporte nativo para workloads Kubernetes.
- Alertas dimensionáveis: alertas mais rápidos e com menor latência.
- Nova experiência de Workbooks: mais responsiva e com suporte a GitHub.
- Monitoramento multicloud: suporte a AWS e GCP via Azure Arc.

 Exemplo de Configuração: Monitorar uma VM Linux
1 Criar um Log Analytics Workspace:

az monitor log-analytics workspace create \
  --resource-group MeuGrupo \
  --workspace-name logWorkspace

2 - Ativar diagnóstico na VM:

az monitor diagnostic-settings create \
  --resource-id /subscriptions/<id>/resourceGroups/MeuGrupo/providers/Microsoft.Compute/virtualMachines/minhaVM \
  --workspace logWorkspace \
  --name "diagVM" \
  --logs '[{"category": "Syslog", "enabled": true}]'

3 - Criar alerta de CPU alta:

az monitor metrics alert create \
  --name "AlertaCPUAlta" \
  --resource-group MeuGrupo \
  --scopes /subscriptions/<id>/resourceGroups/MeuGrupo/providers/Microsoft.Compute/virtualMachines/minhaVM \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action-groups meuGrupoDeAcoes

Visualização e Análise

- Workbooks: dashboards com gráficos, tabelas e filtros.
- KQL (Kusto Query Language): linguagem para consultar logs.

Perf
| where ObjectName == "Processor"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)

Boas Práticas e Pontos de Atenção

- Custo: o maior custo vem da ingestão e retenção de logs. Use filtros e retenção inteligente.
- Segurança: habilite RBAC e criptografia nos workspaces.
- Governança: use Azure Policy para forçar a ativação do monitoramento.
- Escalabilidade: use autoscale com base em métricas de uso.
- Alertas: combine alertas com Logic Apps para automações.

1. Workbook Personalizado – Exemplo: Monitoramento de CPU de VMs
   
Este exemplo exibe a média de uso de CPU por VM nos últimos 30 minutos:

{
  "name": "Workbook-CPU-VMs",
  "location": "brazilsouth",
  "properties": {
    "displayName": "Uso de CPU por VM",
    "category": "Desempenho",
    "serializedData": "{\"items\":[{\"type\":1,\"content\":\"Uso médio de CPU por VM nos últimos 30 minutos\"},{\"type\":3,\"content\":\"Perf | where ObjectName == 'Processor' and CounterName == '% Processor Time' | summarize avg(CounterValue) by bin(TimeGenerated, 5m), Computer | render timechart\"}]}",
    "sourceId": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.OperationalInsights/workspaces/<workspaceName>"
  }
}

Painel de Alertas – Exemplo: Alerta de CPU Alta
🔔 Alerta baseado em métrica (CPU > 80%)
resource cpuAlert 'Microsoft.Insights/metricAlerts@2023-04-03' = {
  name: 'AlertaCPUAlta'
  location: 'global'
  properties: {
    description: 'Alerta quando CPU > 80%'
    severity: 2
    enabled: true
    scopes: [
      vm.id
    ]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      allOf: [
        {
          name: 'cpuCrit'
          metricName: 'Percentage CPU'
          metricNamespace: 'Microsoft.Compute/virtualMachines'
          operator: 'GreaterThan'
          threshold: 80
          timeAggregation: 'Average'
        }
      ]
    }
    autoMitigate: true
  }
}



🧱 3. Modelo Bicep – Criação de Workspace + Workbook + Alerta
param location string = 'brazilsouth'
param workspaceName string = 'logWorkspace'

resource workspace 'Microsoft.OperationalInsights/workspaces@2021-12-01-preview' = {
  name: workspaceName
  location: location
  properties: {
    sku: {
      name: 'PerGB2018'
    }
    retentionInDays: 30
  }
}

resource workbook 'Microsoft.Insights/workbooks@2023-06-01' = {
  name: 'Workbook-CPU-VMs'
  location: location
  properties: {
    displayName: 'Uso de CPU por VM'
    category: 'Desempenho'
    sourceId: workspace.id
    serializedData: '{"items":[{"type":1,"content":"Uso médio de CPU por VM nos últimos 30 minutos"},{"type":3,"content":"Perf | where ObjectName == \'Processor\' and CounterName == \'% Processor Time\' | summarize avg(CounterValue) by bin(TimeGenerated, 5m), Computer | render timechart"}]}'
  }
}

2. Painel de Alertas – Exemplo: Alerta de CPU Alta
🔔 Alerta baseado em métrica (CPU > 80%)
resource cpuAlert 'Microsoft.Insights/metricAlerts@2023-04-03' = {
  name: 'AlertaCPUAlta'
  location: 'global'
  properties: {
    description: 'Alerta quando CPU > 80%'
    severity: 2
    enabled: true
    scopes: [
      vm.id
    ]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      allOf: [
        {
          name: 'cpuCrit'
          metricName: 'Percentage CPU'
          metricNamespace: 'Microsoft.Compute/virtualMachines'
          operator: 'GreaterThan'
          threshold: 80
          timeAggregation: 'Average'
        }
      ]
    }
    autoMitigate: true
  }
}



🧱 3. Modelo Bicep – Criação de Workspace + Workbook + Alerta
param location string = 'brazilsouth'
param workspaceName string = 'logWorkspace'

resource workspace 'Microsoft.OperationalInsights/workspaces@2021-12-01-preview' = {
  name: workspaceName
  location: location
  properties: {
    sku: {
      name: 'PerGB2018'
    }
    retentionInDays: 30
  }
}

resource workbook 'Microsoft.Insights/workbooks@2023-06-01' = {
  name: 'Workbook-CPU-VMs'
  location: location
  properties: {
    displayName: 'Uso de CPU por VM'
    category: 'Desempenho'
    sourceId: workspace.id
    serializedData: '{"items":[{"type":1,"content":"Uso médio de CPU por VM nos últimos 30 minutos"},{"type":3,"content":"Perf | where ObjectName == \'Processor\' and CounterName == \'% Processor Time\' | summarize avg(CounterValue) by bin(TimeGenerated, 5m), Computer | render timechart"}]}'
  }
}

Painel de Alertas – Exemplo: Alerta de CPU Alta
🔔 Alerta baseado em métrica (CPU > 80%)
resource cpuAlert 'Microsoft.Insights/metricAlerts@2023-04-03' = {
  name: 'AlertaCPUAlta'
  location: 'global'
  properties: {
    description: 'Alerta quando CPU > 80%'
    severity: 2
    enabled: true
    scopes: [
      vm.id
    ]
    evaluationFrequency: 'PT1M'
    windowSize: 'PT5M'
    criteria: {
      allOf: [
        {
          name: 'cpuCrit'
          metricName: 'Percentage CPU'
          metricNamespace: 'Microsoft.Compute/virtualMachines'
          operator: 'GreaterThan'
          threshold: 80
          timeAggregation: 'Average'
        }
      ]
    }
    autoMitigate: true
  }
}



🧱 3. Modelo Bicep – Criação de Workspace + Workbook + Alerta
param location string = 'brazilsouth'
param workspaceName string = 'logWorkspace'

resource workspace 'Microsoft.OperationalInsights/workspaces@2021-12-01-preview' = {
  name: workspaceName
  location: location
  properties: {
    sku: {
      name: 'PerGB2018'
    }
    retentionInDays: 30
  }
}

resource workbook 'Microsoft.Insights/workbooks@2023-06-01' = {
  name: 'Workbook-CPU-VMs'
  location: location
  properties: {
    displayName: 'Uso de CPU por VM'
    category: 'Desempenho'
    sourceId: workspace.id
    serializedData: '{"items":[{"type":1,"content":"Uso médio de CPU por VM nos últimos 30 minutos"},{"type":3,"content":"Perf | where ObjectName == \'Processor\' and CounterName == \'% Processor Time\' | summarize avg(CounterValue) by bin(TimeGenerated, 5m), Computer | render timechart"}]}'
  }
}



