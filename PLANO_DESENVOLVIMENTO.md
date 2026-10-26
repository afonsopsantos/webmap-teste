# Plano de Desenvolvimento do Geoportal WebGIS - Dados de Vicosa/MG

## Visão Geral
Webmap interativo utilizando as camadas GeoJSON do projeto, com base Leaflet, mapas de fundo OpenStreetMap e satélite, e todas as funcionalidades solicitadas: visualização de camadas, atributos, medição, pesquisa, zoom, geolocalização, exportação e impressão.

---

## 1. ARQUITETURA E ARQUIVOS DO PROJETO

### 1.1 Estrutura de Arquivos
```
/geoportal_vicosa.html          ← Ponto de entrada (já criado)
/geojson/                       ← Dados fontes (já existentes)
  - Vicosa_Municipio.geojson   ← Limite municipal (polígono)
  - Vicosa_Distritos.geojson   ← Distritos administrativos (polígonos)
  - Vicosa_Escolas.geojson     ← Escolas (pontos - 59 features)
  - Vicosa_Vilas.geojson       << Vilas/ruralidades (pontos - 3 features)
  - Vicosa_CARimoveis.geojson  << Imóveis rurais (polígonos - ~100 features de Viçosa)
/styles.css                     ← Estilos customizados (opcional, inline no HTML)
/images/                        ← Ícones/logotipos (opcional)
```

### 1.2 Dependências Externas (CDN)
- Leaflet 1.9.4 (CSS + JS)
- Leaflet.Search 2.10.1
- Leaflet.Measure 1.0.3
- Leaflet.Control.Print 0.1.0
- Font Awesome 6.4.0 (ícones)

---

## 2. MAPAS DE BASE

| Camada | URL | Atributos |
|--------|-----|-----------|
| **OpenStreetMap** | `https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png` | &copy; OpenStreetMap contributors |
| **Satélite** | `https://{s}.tile.stamen.com/termer/{z}/{x}/{y}.jpg` | Stamen Maps & OSM contributors |

**Controle de camadas**: Incluído automaticamente via `L.control.baseLayers()`, permitindo alternar entre os dois mapas.

---

## 3. CAMADAS VETORIAIS E SEU TRATAMENTO

### 3.1 Camada: Município (Vicosa_Municipio.geojson)
- **Tipo**: Polígono (MultiPolygon)
- **Dados relevantes**: NOME, GEOCODIGO, AREA, LEGENDA, dados históricos
- **Estilo**: Contorno cor #2c3e50 (azul escuro), fill-opacity 0.1
- **Popup**: Mostra nome, código, área (300.15 km²), ano de fundação (1871), lei de criação
- **Funcionalidade**: Visibilidade toggle, clique para popup informativo

### 3.2 Camada: Distritos (Vicosa_Distritos.geojson)
- **Tipo**: Polígonos (vários distritos)
- **Dados relevantes**: NOME, GEOCODIGO, AREA, LEICRIACAO, DATAIOMG
- **Estilo**: Contorno cor #3498db (azul), fill-opacity baixa
- **Popup**: Nome do distrito, código, área em km², lei de criação
- **Funcionalidade**: Poderá ser ocultado para melhor performance com muitos dados

### 3.3 Camada: Vilas (Vicosa_Vilas.geojson)
- **Tipo**: Pontos (3 features)
- **Dados relevantes**: NOME, GEOCODIGO, LAT, LONG
- **Estilo**: Marcadores com ícone padrão Leaflet (cor variando por categoria)
- **Popup**: Nome da vila e código
- **Funcionalidade**: Ideal para zoom regional, clique destaca local

### 3.4 Camada: Escolas (Vicosa_Escolas.geojson)
- **Tipo**: Pontos (59 features)
- **Atributos importantes**:
  - ESCHOla (nome da escola)
  - COD_INEP (código oficial)
  - CATEGORIA (Pública/Privada)
  - MOD_ENSINO (Fundamental, Médio, etc.)
  - PORT (número de portarias)
  - TEL (telefone)
  - LOCAL_ (urban/rural)
  - REG_EDUC (Sim/Não)
- **Estilo**: Marcadores azuis para públicas, vermelhas para privadas
- **Popup detalhado**: Contém todos os campos acima em formato legível
- **Funcionalidade**: Pesquisa textual, destaque, popup clicável

### 3.5 Camada: CAR Imóveis (Vicosa_CARimoveis.geojson)
- **Tipo**: Polígonos (muitas features - filtrado apenas Viçosa)
- **Atributos importantes**:
  - cod_imovel (identificador único)
  - num_area (área em hectares)
  - ind_status (AT = Aguardando análise, PE = Processo)
  - ind_tipo (IRU - Interior Rural Urbano)
  - des_condic (condição do imóvel)
  - dat_criaca, dat_atuali (Datas de criação/atualização)
- **Estilo**: Fill color baseado em status (verde = AT, laranja = PE, cinza = outros)
- **Popup**: Código do imóvel, área, status, tipo, condição
- **Funcionalidade**: Filtro por município (apenas Viçosa), tooltip com atributos chave

---

## 4. FUNCIONALIDADES IMPLEMENTADAS

### 4.1 Habilitar Visualização de Camadas ✅
- **Implementação**: `L.control.layers(baseLayers, overlayLayers)`
- **Interface**: Painel lateral esquerdo com checkboxes
- **Grupo de camadas**: Municipal, Distritos, Vilas, Escolas, CAR Imóveis
- **Mapas base**: Toggle entre OSM e Satélite no canto superior esquerdo

### 4.2 Visualizar Atributos das Camadas ✅
- **Popups**: Cada feature tem popup com dados relevantes
- **Escolas**: Popup completo com 8+ campos (nome, INEP, categoria, modalidade, portarias, telefone, localização)
- **CAR Imóveis**: Popup com código, área, status, tipo, condição
- **Município/Distritos**: Popup com dados históricos

### 4.3 Medir Distâncias e Áreas ✅
- **Plugin**: `L.Control.Measure`
- **Unidades**: km/m para distância, ha/sqkm para área
- **Interface**: Dois botões na control-panel
- **Modo**: Desenhar linha/área no mapa, resultados em tempo real

### 4.4 Pesquisar nos Atributos das Camadas ✅
- **Plugin**: `L.Control.Search`
- **Campo de busca**: Na control-panel superior direita
- **Propriedades pesquisáveis**: ESCHOla, COD_INEP, MUN (de escolas)
- **Resultados**: Marcador no mapa, popup aberto, destaque visual
- **Limpar**: Botão "Limpar busca" na interface

### 4.5 Aumentar e Diminuir Nível de Zoom ✅
- **Controles nativos**: Roda do mouse, botões +/- da Leaflet
- **Controle de zoom**: Padrão da LeafTop no canto direito
- **Zoom ótimo**: Inicial em zoom 11 (visão municipal), pode zoom até 19 (ruas)

### 4.6 Geolocalizar ✅
- **Plugin**: `L.locate()` 
- **Botão**: "Localizar-me" no canto inferior direito
- **Funcionalidade**: 
  - Solicita permissão do navegador
  - Centraliza mapa na posição do usuário
  - Desenha círculo de precisão
  - Mensagem de erro se negado

### 4.7 Exportar Camadas ✅
- **Botão**: "Exportar" na control-panel
- **Funcionalidade**: 
  - Gera arquivo JSON com features das escolas selecionadas
  - Download automático do arquivo `vicosa_layers_export.json`
  - Mantém atributos completos

### 4.8 Imprimir Mapa ✅
- **Plugin**: `L.Control.Print`
- **Configurações**: 
  - Título: "Imprimir Mapa"
  - Posição: canto superior direito
  - Dimensões: 800x600 (padrão) ou A4/A3
  - Remove "Powered by Leaflet"
- **Funcionalidade**: Gera PDF/imagem do mapa visível

---

## 5. PLANO DE DESENVOLVIMENTO FASEADO

### FASE 1: Estrutura Base e Mapas de Base (1 semana)
| Tarefa | Responsável | Status |
|--------|-------------|--------|
| 1.1 Criar estrutura HTML básica | Desenvolvedor | ✅ Concluído |
| 1.2 Incluir CDNs Leaflet e plugins | Desenvolvedor | ✅ Concluído |
| 1.3 Configurar camadas base OSM + Satélite | Desenvolvedor | ✅ Concluído |
| 1.4 Adicionar controle de base layers | Desenvolvedor | ✅ Concluído |
| 1.5 Testar carregamento inicial no navegador | QA | ⏳ Pendente |

### FASE 2: Camadas Vetoriais e Estilização (1 semana)
| Tarefa | Responsável | Status |
|--------|-------------|--------|
| 2.1 Carregar Vicosa_Municipio.geojson | Desenvolvedor | ✅ Concluído |
| 2.2 Configurar estilo e popup municipal | Desenvolvedor | ✅ Concluído |
| 2.3 Carregar Vicosa_Distritos.geojson | Desenvolvedor | ✅ Concluído |
| 2.4 Aplicar estilo distritos e popups | Desenvolvedor | ✅ Concluído |
| 2.5 Carregar Vicosa_Vilas.geojson | Desenvolvedor | ✅ Concluído |
| 2.6 Estilizar marcadores de vilas | Desenvolvedor | ✅ Concluído |
| 2.7 Carregar Vicosa_Escolas.geojson (59 features) | Desenvolvedor | ✅ Concluído |
| 2.8 Implementar ícones cor/ tipo por categoria | Desenvolvedor | ✅ Concluído |
| 2.9 Criar popups detalhados por escola | Desenvolvedor | ✅ Concluído |
| 2.10 Carregar Vicosa_CARimoveis.geojson (filtrado Viçosa) | Desenvolvedor | ✅ Concluído |
| 2.11 Filtrar apenas município Viçosa (~20 features) | Desenvolvedor | ✅ Concluído |
| 2.12 Estilizar polígonos por status (AT/PE) | Desenvolvedor | ✅ Concluído |
| 2.13 Criar popups para imóveis | Desenvolvedor | ✅ Concluído |
| 2.14 Testar performance com todas camadas | QA | ⏳ Pendente |

### FASE 3: Controle de Camadas e Interface (3-4 dias)
| Tarefa | Responsável | Status |
|--------|-------------|--------|
| 3.1 Adicionar painel lateral de camadas | Desenvolvedor | ✅ Concluído |
| 3.2 Criar painel de busca por escolas | Desenvolvedor | ✅ Concluído |
| 3.3 Criar painel de medição (distância/área) | Desenvolvedor | ✅ Concluído |
| 3.4 Adicionar botão de geolocalização | Desenvolvedor | ✅ Concluído |
| 3.5 Criar painel de impressão | Desenvolvedor | ✅ Concluído |
| 3.6 Criar painel de exportação | Desenvolvedor | ✅ Concluído |
| 3.7 Estilizar todos os painéis (CSS custom) | Desenvolvedor | ✅ Concluído |
| 3.8 Testar responsividade móvel | QA | ⏳ Pendente |

### FASE 4: Funcionalidades Avançadas (3 dias)
| Tarefa | Responsável | Status |
|--------|-------------|--------|
| 4.1 Implementar busca textual em escolas | Desenvolvedor | ✅ Concluído |
| 4.2 Funcionalidade "Limpar busca" | Desenvolvedor | ✅ Concluído |
| 4.3 Testar medição de distância entre pontos | QA | ⏳ Pendente |
| 4.4 Testar medição de área de terrenos | QA | ⏳ Pendente |
| 4.5 Testar geolocalização em diferentes navegadores | QA | ⏳ Pendente |
| 4.6 Testar impressão em diferentes formatos | QA | ⏳ Pendente |
| 4.7 Testar exportação JSON | QA | ⏳ Pendente |

### FASE 5: Testes, Deploy e Documentação (4-5 dias)
| Tarefa | Responsável | Status |
|--------|-------------|--------|
| 5.1 Testes em navegadores (Chrome, Firefox, Edge, Safari) | QA | ⏳ Pendente |
| 5.2 Testes em dispositivos móveis (telular/tablet) | QA | ⏳ Pendente |
| 5.3 Otimizar performance (carregamento de ~100 features) | Desenvolvedor | ⏳ Pendente |
| 5.4 Ajustar CSS para melhor aparência profissional | Desenvolvedor | ⏳ Pendente |
| 5.5 Preparar arquivo para upload | Desenvolvedor | ⏳ Pendente |
| 5.6 Deploy em hospedagem gratuita (GitHub Pages/Netlify) | DevOps | ⏳ Pendente |
| 5.7 Verificar funcionalidades no ambiente production | QA | ⏳ Pendente |
| 5.8 Criar documentação rápida de uso | Documentação | ⏳ Pendente |

---

## 6. HOSPEDAGEM GRATUITA

### Opção A: GitHub Pages (Recomendada)
1. Criar repositório no GitHub: `vicosa-geoportal`
2. Enviar arquivos:
   - `geoportal_vicosa.html`
   - Pasta `geojson/` (todos os arquivos)
3. Ativar GitHub Pages:
   - Settings → Pages
   - Source: `main` branch / pasta `/(root)`
4. Aguardar build (1-2 minutos)
5. URL: `https://seu-usuario.github.io/vicosa-geoportal/`

### Opção B: Netlify
1. Acesse netlify.com e crie conta gratuita
2. Arraste a pasta do projeto para interface
3. ou conecte repositório GitHub
4. Site gerado automaticamente em URL estilo `seu-site.netlify.app`

### Opção C: Vercel
1. Importar repositório Git
2. Configurar como "Static HTML"
3. Deploy instantâneo em URL `seu-site.vercel.app`

---

## 7. OTIMIZAÇÕES PARA DADOS GRANDES

### 7.1 CAR ImÓVEIS (muitos polígonos)
- **Filtro client-side**: Apenas municípios selecionados são carregados
- **Simplificação geométrica**: Se necessário, usar `L.simplify()` nos polígonos
- **Agrupamento**: Mostrar apenas nível de distrito/município inicialmente

### 7.2 MEMÓRIA E PERFORMANCE
- **Carregamento preguiçoso**: Camadas de distritos e CAR carregam após mapa inicial
- **Remover layer**: `map.removeLayer(camada)` quando não necessário
- **Atualizar bounds**: `map.fitBounds()` após carregar dados grandes

### 7.3 TESTES DE PERFORMANCE
- Testar com 59 escolas + 20 CAR + 1 município + 3 vilas
- Verificar fps de rolagem e tempo de resposta de cliques
- Confirmar que zoom/pan é fluido em dispositivos móveis

---

## 8. ACESSIBILIDADE E USABILIDADE

### 8.1 Contraste e Cores
- Cores escolhidas têm contraste adequado (#2c3e50, #3498db, branco)
- Texto nos popups legível sobre diferentes fundos de mapa
- Cores de status (verde/laranja) são distinguíveis para daltónicos

### 8.2 Controles Intuitivos
- Ícones familiares (lupa, localização, impressão, download)
- Textos em português (Brasil)
- Ordem lógica: mapas base → camadas → busca → medir → imprimir
- Feedback visual em hover e clique

### 8.3 Responsividade
- Layout se adapta de 320px (celular) a 1920px+ (desktop)
- Controles se reorganizam em coluna única em celulares
- Fontos aumentam em dispositivos menores

---

## 9. BUGS CONHECIDOS E LIMITAÇÕES

1. **CAR Imóveis de outros municípios**: Arquivo contém dados de Coimbra, Teixeiras etc. O código filtra apenas Viçosa - remova o filtro para carregar todos.
2. **Popup em português com acentos**: Arquivos GeoJSON têm acentos codificados (Viçosa → Vi�osa). Solução: garantir encoding UTF-8 (já feito no HTML).
3. **Busca apenas em escolas**: Plugin Search configurado apenas para a layer de escolas. Pode estender para outros layers.
4. **Impressão**: Richer em alguns navegadores móveis - testar antes de entregar ao usuário.

---

## 10. PRÓXIMOS PASSOS IMEDIATOS

1. ✅ **Concluído**: Criar arquivo HTML com todas as funcionalidades
2. ⏳ **Próximo**: Testar arquivo localmente abrindo no navegador
3. ⏳ **Posterior**: Subir para GitHub Pages ou Netlify
4. ⏳ **Posterior**: Coletar feedback sobre usabilidade e fazer ajustes

---

**Arquivos gerados:**
- `C:\webmap\deepseek\geoportal_vicosa.html` - Webmap completo
- `C:\webmap\deepseek\geojson/` - 5 arquivos de dados GeoJSON

**Próximo ação**: Abrir o arquivo `geoportal_vicosa.html` em um navegador para validar que todas as camadas carregam corretamente e as funcionalidades (busca, medição, geolocalização) funcionam como esperado.