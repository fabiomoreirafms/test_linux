**Modelo Bomb**

1. **Player::runTimeCriticalRoutines(Phase 3)**

External application (*Node*) faz a chamada do método
'runTimeCriticalRoutines' da classe *WorldModel* (que estende
*Simulation*). Esse método, que representa um frame da simulação, dá
origem a quatro séries de chamadas aos métodos de mesmo nome dos
*Players*; cada série representa uma phase do frame: 'dynamics',
'transmit', 'receive' e 'process'. O pseudo-código dessa sequência de
chamadas é fornecido a seguir.

| WorldModel::runTimeCriticalRoutines()
| {
|     **for** (i in 0:3)
|     {
|         phase = i;
|         **for** (each Player)
|         {
|             Player.runTimeCriticalRoutines();
|         } 
|     }
|    
|     frame += 1;
| }
|     
| Player::runTimeCriticalRoutines()
| {
|     **for** (each System)
|     {
|         System.runTimeCriticalRoutines();
|     } 
| }

Cada *System*, ao receber uma chamada ao método
'runTimeCriticalRoutines', em função da fase corrente definida pela
classe *WorldModel*, irá chamar o método correspondente de sua
estrutura: 'dynamics', 'transmit', 'receive' e 'process'.

O Player repassa a chamada de 'runTimeCriticalRoutines' para o Agent, no
momento que a fase 3 é ativada.

2. **Agent::runTimeCriticalRoutines(Phase 3)**

Este método abstrato sobrepõe a implementação feita na classe herdada,
realizando as seguintes ações:

- Chamada ao método 'state': esse método faz o preenchimento da
estrutura de dados 'FighterState ';

- Chamada ao método 'reasoning': a partir das variáveis da estrutura
'FighterState', realiza um processamento com base em algoritmos e
estruturas de Inteligência Artificial, que resulta no preenchimento de
outra estrutura de dados 'FighterAction';

- Chamada ao método 'action': irá acionar os métodos respectivos para
executar as ações determinadas pelo método anterior, por meio do
conteúdo da estrutura 'FighterAction'.

3. **Agent::action()**

Chama o método:

- 'actionReleaseBomb', se 'hasBombReleaseAction' for 'true'.

a. **ActionReleaseBomb()**

O método 'actionReleaseBomb' realiza as seguintes ações:

1 - verifica se realmente há condições para que o detonador ('fuze') da
bomba seja armado (ver atributos 'arming', 'fuzeAltitude' e 'fuzeTime'
da classe Bomb).

2 - se as condições de armação do detonador forem satisfeitas, realiza o
seguinte:

2.1 - chama o método 'triggerEvent' no Player com o parâmetro
RELEASE_BOMB. A intenção é acionar o método 'handleReleaseBombEvent' da
classe Player, que tratará do lançamento das bombas.

2.2 - chama o método 'triggerEvent' no Fixed Ground Target com o
parâmetro ATTACK. A intenção é chamar o método 'handleAttackEvent' do
alvo atacado (classe Fixed Ground Target), fornecendo uma estrutura
Attack Data como parâmetro;

2.3 - atualiza o peso da aeronave, descontando o peso das bombas
lançadas (atributo 'weight' de Bomb e atributo 'grossWeight' de
DynamicsModel).

4. **Component::Player::triggerEvent(RELEASE_BOMB)**

O método 'triggerEvent' permite que um Component possa deflagrar um
evento em outro Component, especificando o tipo de evento por meio do
parâmetro 'eventType'. Neste caso, o tipo de evento é RELEASE_BOMB. Este
método aciona o método 'handleReleaseBombEvent'.

O método retorna true caso o evento tenha sido tratado, ou false caso
contrário.

5. **Player::handleReleaseBombEvent()**

O método handleReleaseBombEvent(tgtGnd:Fixed Ground Target) trata o
evento de disparo de uma bomba em um alvo terrestre, tornando
disponíveis para o Fixed Ground Target os dados para preenchimento de
Attack Data.

6. **Weapon::runTimeCriticalRoutines(Phase 0)**

Todas as bombas herdam de Weapon, que, por sua vez, herda de Player.
Assim, uma bomba possui um método 'runTimeCriticalRoutines' herdado da
classe Weapon. Em phase 0, os procedimentos realizados por esse método
são os seguintes:

-  Se 'mode' estiver definido como 'pre-released' (armamento acabou de
   ser lançado), chama o método 'atReleaseInit' (sem efeito na classe
   Bomb) e define atributo 'mode' como "active";

-  Chamada ao método 'updateDynamics' da classe Weapon\ *.*

7. **Weapon::atReleaseInit()**

Sem efeito na classe Bomb.

8. **Weapon::updateDynamics()**

Este método sobrepõe a implementação do método 'updatedynamics' na
classe Player\ *,* realizando as seguintes ações:

-  Chamada ao método 'weaponGuidance' da bomba;

-  Chamada ao método 'weaponDynamics' da bomba;

-  Chamada ao método 'updateDynamics' da classe Player.

Os métodos de guiamento e dinâmica da bomba são responsáveis por definir
os valores de algumas variáveis de estado (velocidade do corpo, posições
e velocidade angulares); o método 'updateDynamics' da classe Player, por
sua vez, usa as informações do estado corrente para definir a nova
posição do armamento.

9. **Bomb::weaponGuidance()**

Se a bomba for de 'category' "*guided*" (ver classe *Weapon*), utiliza
as predições fornecidas pelo método 'impactPrediction' para calcular
comandos de aceleração (azimute e elevação) a fim de guiar o armamento
em direção ao alvo. Caso contrário, não define nenhuma diretiva de
guiamento. Para detalhes do algoritmo de guiamento, consulte os códigos
em R fornecidos juntamente com a documentação dos modelos.

10. **Bomb::impactPrediction()**

Realiza uma previsão do ponto de impacto da bomba a partir das condições
iniciais fornecidas como parâmetros do método: posição corrente no
frame; velocidade corrente no frame; altitude do alvo, passo de
integração a ser usado nos cálculos e tempo máximo de voo. O algoritmo
considera os efeitos da gravidade e do arrasto ao realizar a estimativa.
Para maiores detalhes, consulte os códigos em R fornecidos juntamente
com a documentação dos modelos.

11. **Bomb::weaponDynamics()**

O método 'weaponDynamics' define o estado da bomba (ângulos de Euler,
velocidades lineares e angulares), considerando o efeito da gravidade,
do arrasto, e de acelerações do sistema de guiamento (se houver). Para
detalhes do algoritmo de dinâmica do corpo, consulte os códigos em R
fornecidos juntamente com a documentação dos modelos.

12. **Player::setEulerAngles()**

Define o valor do atributo 'eulerAngles' da classe Player.

13. **Player::setAngularVelocities()**

Define o valor do atributo 'angularVel' da classe Player.

14. **Player::setLinearVelocities()**

Define o valor do atributo 'velocity' da classe Player.

15. **Player::updateDynamics()**

Este método usa as informações de estado (posições angulares,
velocidades lineares e angulares) definidos pelo método ‘weaponDynamics’
para calcular a nova posição da bomba.

16. **Weapon::runTimeCriticalRoutines(Phase 3)**

Por se tratar da última phase do frame, chama o método 'updateTOF' para
que o tempo de voo do armamento (tof) seja atualizado.

17. **Weapon::updateTOF()**

Atualiza o valor do atributo 'tof'.

Se o valor de 'tof' for superior ao de 'maxTOF', muda o atributo 'mode'
(ver classe Player) para "detonated".

18. **Component::Fixed Ground Target::triggerEvent(ATTACK)**

O Component Agent deflagra o evento ATTACK no Component Fixed Ground
Target por meio do método 'triggerEvent'. Este método chama o método
'handleAttackEvent'.

O método retorna true caso o evento tenha sido tratado, ou false caso
contrário.

19. **Fixed Ground Target::handleAttackEvent()**

Esse método permite inserir a estrutura de dados do ataque na lista
'attacks'. O ataque será processado a partir do método
'runNonTimeCriticalRoutines'.

20. **Fixed Ground Target::runNonTimeCriticalRoutines(Phase 3)**

Este método calcula o 'damage' do ataque realizado. Para isso, primeiro
verifica se há algum ataque a ser processado (compara atributos
'releaseTime' e 'timeOfFlight' com a hora da simulação).

Fixed Ground Target é um POI e sua posição pode ser definida pelos
atributos 'latitude', 'logintude' e 'altitude'. Para definir seu tamanho
espacial, o atributo 'vertices' contém 'azimuth', 'distance' e a
quantidade de vértices para definição do polígono (no mínimo 3 são
requeridos).

O processamento de um ataque consiste dos seguintes passos:

-  o atributo 'circularError' (baseado em estatísticas) associa erros de
   pontaria do piloto e/ou erros do sistema de pontaria da aeronave. Já
   o atributo 'axisOfAttack' define a direção de voo da aeronave no
   momento do ataque. Com a direção do ataque e o erro circular médio é
   possível determinar a posição da bomba central que foi lançada. O
   atributo 'ripple' faz a distribuição das bombas ao longo desse ponto
   central respeitando a distância entre elas. Caso o número de bomba
   seja um número par, a posição central será definida como a média das
   duas bombas centrais. Tais atributos são da classe Attack Data, um
   Object relacionado a Fixed Ground Target;

-  para cada impacto, estima o percentual de dano causado (usa método de
   Monte Carlo, sorteando pontos nas proximidades do alvo e verifica a
   proporção deles que são interiores ao polígono que define o alvo e ao
   círculo definido pela área média de eficácia do armamento - consulta
   o banco de dados 'tgtDamageDB' da Simulation). O polígono definido
   precisa ser maior que o tamanho espacial do alvo que está sendo
   analisado.
