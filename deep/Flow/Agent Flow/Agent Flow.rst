**Agent Flow**

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
| **for** (i **in** 0:3)
| {
| phase = i;
| **for** (**each** Player)
| {
| Player.runTimeCriticalRoutines();
| } 
| }
| frame += 1;
| }
| Player::runTimeCriticalRoutines()
| {
| **for** (**each** **System**)
| {
| **System**.runTimeCriticalRoutines();
| } 
| }

2. **Agent::runTimeCriticalRoutines(Phase 3)**

Semelhantemente, o *Player* repassa a chamada de
'runTimeCriticalRoutines' para o *Agent*, no momento que a fase 3 é
ativada.

A partir disso, a classe Agent sobrepõe a implementação original de
Player, realizando as seguintes ações na fase 3 ('process'):

Chamada ao método 'state';

Chamada ao método 'reasoning';

Chamada ao método 'action'.

3. **Agent::state()**

Preenche os atributos da classe *Agent State*, a partir de dados
disponíveis na simulação, tratando-os conforme a necessidade\ *.*

Por exemplo, a partir da classe *Simulation*, atribui valores às
variáveis:

- phase;

- simExecTime;

- simTimeOfDay.

Outros exemplos a partir da classe *Player*, são:

- latitude;

- longitude;

- altitude;

- position;

- velocity;

- gndSpeed;

- gndTrack;

- accel;

- eulerAngles;

- terrainElev;

- damage.

4. **Agent::reasoning()**

Define valores para os atributos da classe *Agent Action*.

É onde os métodos principais de inteligência artificial devem ser
implementados, a fim de determinar quais serão as ações (*Agent Action*)
que o *Agent* deve tomar a partir de seu estado (*Agent State*).

Método abstrato; deve ser implementado pelas classes herdeiras.

5. **Agent::action()**

Atualiza atributos e/ou chama métodos de classes (ex: *System*) tendo
por referência os atributos da classe *Agent Action.*

Método abstrato; deve ser implementado pelas classes herdeiras.
