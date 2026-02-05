# 📊 Painel de Entrega de Resultados Analistas

Este painel tem como objetivo monitorar e detalhar a performance de entregas dos analistas, permitindo uma visão estratégica (Gerencial) e tática (Operacional) sobre prazos, qualidade e volume de entregas.

## 🔄 Fluxo de Dados

O painel consome dados de uma lista SharePoint e os processa para visualização.

```mermaid
graph TD
    A[SharePoint List<br>MateriaisdeEntregadeResultados-2025] -->|Import Mode| B[Power BI Dataset]
    B --> C{Transformação Power Query<br>(M Language)}
    C --> D[Tabela Fato:<br>ControleEntregaResultados]
    C --> E[Dimensões Auxiliares]

    subgraph Data Model [Star Schema Simplificado]
        D -- Cliente --> F[Dim_Frequencia_Clientes]
        D -- DataDaProximaEntrega --> G[Dim_Calendario]
    end

    subgraph Lógica de Negócio [DAX Measures]
        D --> H[KPI: Entregas Atrasadas]
        D --> I[KPI: Taxa de Realização]
    end

    Data Model --> J[Report Views]
    J --> K[Pág: Detalhamento Geral]
    J --> L[Pág: Painel de Indicadores]
```

## 📐 Modelo Semântico e Tabelas

### 1. `ControleEntregaResultados` (Fato)

Tabela central que registra cada entrega agendada ou realizada.

- **Granularidade**: Uma linha por entrega por cliente/analista.
- **Campos Críticos**:
  - `DataDaProximaEntrega`: Define o SLA.
  - `UltimaEntrega`: Data efetiva da realização. Se `null`, está pendente.

### 2. `Dim_Frequencia_Clientes`

Define a regra de contratual de cada cliente.

- **Propósito**: Calcular a meta (`Target`) de entregas anuais.
- **Regra**: Se cliente é "Mensal", meta = 12. Se "Trimestral", meta = 4.

### 3. `Dim_Calendario`

Tabela de datas padrão para permitir filtros temporais (Ano, Mês).

## 🧮 Principais Medidas (DAX)

### `Entregas Atrasadas`

Calcula quantas entregas estão pendentes e fora do prazo.

- **Lógica**: Considera atrasado apenas se o mês de referência já encerrou (Tolerância M-1).

```dax
CALCULATE(
    COUNTROWS(ControleEntregaResultados),
    ControleEntregaResultados[DataDaProximaEntrega] < TODAY(),
    ISBLANK(ControleEntregaResultados[UltimaEntrega]) || ...
)
```

### `Taxa de Realização %`

Métrica de eficiência.

```dax
DIVIDE( [Entregas Realizadas], [Entregas Previstas Ano] )
```

## 🎨 Detalhes Técnicos do Frontend

### Padrão Visual (Branding)

- **Cores**: Vermelho Edenred (`#E20613`) para destaques e KPIs.
- **Fontes**: `DIN` para títulos e números grandes. `Segoe UI` para leitura.
- **Layout**: Headers padronizados com logo à esquerda e filtros agrupados em container "clean" à direita.

### Estrutura de Páginas

1.  **Painel de Indicadores**: Visão macro. Cards de KPI e gráficos de tendência.
2.  **Detalhamento Geral**: Tabela analítica. Filtros por Segmento, Analista e Status.

---

_Gerado automaticamente pelo Agente de Documentação._
