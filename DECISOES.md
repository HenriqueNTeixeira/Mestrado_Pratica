# Decisões do projeto

## Ambiente
| Item | Valor | Justificativa |
|---|---|---|
| ROS | Noetic | imagem lar-gazebo:noetic, já validada |
| Build | `catkin build` | imagem construída com catkin_tools; misturar com catkin_make quebra |
| Mapa | lab_robotica_06mai2019 (2,5 cm/célula) | oficial do LAR, resolução mais fina que o GMapping próprio (5 cm) |
| Área navegável | 37,13 m² (folga 0,35 m) | calculado por erosão do espaço livre |

## Sensor
| Item | Valor | Fonte |
|---|---|---|
| Modelo | SICK LMS1xx | Carvalho et al. (2023), Fig. 13 |
| Feixes | 541 | extraído das bags reais |
| FOV | 270° (-135° a +135°) | extraído das bags reais |
| Alcance | 0,01 – 20 m | extraído das bags reais |

## Geração da base sintética
| Item | Valor | Fonte |
|---|---|---|
| Espaçamento entre poses | 0,25 m (a validar) | Zhao et al.: intervalo ≈ acurácia esperada |
| Intervalo angular | 10° (36 orientações) | Zhao et al. |
| Ruído de treino | ±0,05–0,1 m posição, ±1° orientação | Zhao et al.: evita decorar a grade |
| Leituras inválidas | preencher com zero | Zhao et al. |
| Células desconhecidas | A DECIDIR empiricamente | comparar histograma sintético vs. bags reais |

## Rede
| Item | Valor | Fonte |
|---|---|---|
| T (janela temporal) | Etapa A: T=1 / Etapa B: T=10 | reduz risco; T=1 vs T=10 vira ablação |
| K (hipóteses) | 10 | Filotheou (CBGL) usa k=10 |
| Codificação angular | sin θ / cos θ | Zhou et al. (2019) |
| Split validação | 25% do treino | Zhao et al. |

## Refinamento e avaliação
| Item | Valor | Fonte |
|---|---|---|
| L_c do Perfect Match | 1 m | Pinto et al. (2013) |
| Sucesso (rede isolada) | erro ≤ 12 cm e ≤ 20° | Carvalho et al.: L_dev/A_dev, tolerância do PM |
| Sucesso (rede + PM) | erro < 10 cm e < 5° | Carvalho et al. |
| Ground truth em dados reais | rastreamento por PM | Carvalho et al. |

## Pendências com os orientadores
1. As bags de 07/05/2019 são do Husky físico ou de simulação?
2. Ok usar o truque do DSOM (obstáculos aleatórios na geração, mapa limpo na entrada)?
3. O código do PM do trabalho AG/PSO pode ser reaproveitado?
