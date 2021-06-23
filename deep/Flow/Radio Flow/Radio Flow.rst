**Radio Flow**

1. **Player1::Player::runTimeCriticalRoutines(Phase 3)**

External application (*Node*) faz a chamada do método
'runTimeCriticalRoutines' da classe *WorldModel* (que estende
*Simulation*). Esse método, que representa um frame da simulação, dá
origem a quatro séries de chamadas aos métodos de mesmo nome dos
*Players*; cada série representa uma phase do frame: 'dynamics',
'transmit', 'receive' e 'process'. O pseudo-código dessa sequência de
chamadas é fornecido a seguir.

| WorldModel::runTimeCriticalRoutines()
| {
| **for** (i in 0:3)
| {
| phase = i;
| **for** (each Player)
| {
| Player.runTimeCriticalRoutines();
| }
| }
| frame += 1;
| }
| Player::runTimeCriticalRoutines()
| {
| **for** (each System)
| {
| System.runTimeCriticalRoutines();
| }
| }

Como consequência, cada *Player* irá chamar o mesmo método de cada um de
seus sistemas (classes *System*). Dessa forma, todas as classes
relacionadas a comunicações por datalink serão ativadas.

2. **Agent 1::Agent::runTimeCritical(Phase 3)**

Semelhantemente, o Player repassa a chamada de 'runTimeCriticalRoutines'
para o Agent, no momento que a fase 3 é ativada.

A partir disso, a classe *Agent* sobrepõe a implementação original de
*Player*, realizando as seguintes ações na fase 3 ('process'):

a. Chamada ao método 'state';

b. Chamada ao método 'reasoning';

c. Chamada ao método 'action'.

3. **Agent1::Agent::action()**

O método ‘action’ deve atualizar atributos e/ou chamar métodos de
classes (ex: *System*) tendo por referência os atributos da classe
*Agent* *Action*, que, por sua vez, é atualizada previamente pelo método
'reasoning', o qual utiliza as variáveis preenchidas em *Agent State*
pelo método 'state'.

4. **Component::Player2::Player::triggerEvent(RD_MSG)**

O método 'triggerEvent' permite que um Component possa deflagrar um
evento em outro Component, especificando o tipo de evento por meio do
parâmetro eventType. Neste caso, o tipo de evento é 'RD_MSG'.

O método retorna true caso o evento tenha sido tratado, ou false caso
contrário.

5. **Player2::Player::handleRadioMsgEvent()**

Este método simplesmente repassa a mensagem recebida para o *Radio*
(caso possua o equipamento definido em sua estrutura).

6. **Radio2::Radio::handleRadioMsgEvent()**

Este método realiza as seguintes verificações:

-  se o *ownship* é destinatário;

-  se a frequência de transmissão é coincidente com a frequência
   sintonizada;

-  se a distância entre transmissor e receptor é inferior a 'maxRange'.

Se as condições forem satisfeitas, insere a mensagem na lista
'messages'.

7. **Player2::Player::runTimeCriticalRoutines(Phase 3)**

Similar ao descrito no item 1.

8. **Agent2::Agent::runTimeCritical(Phase 3)**

O Player repassa a chamada de 'runTimeCriticalRoutines' para o Agent, no
momento que a fase 3 é ativada.

Assim como em 2, este método realiza as seguintes ações na fase 3
('process'):

a. Chamada ao método 'state';

b. Chamada ao método 'reasoning';

c. Chamada ao método 'action'.

9. **Agent2::Agent::state()**

A partir de outras classes disponíveis *(*\ ex: *Simulation, Player),*
este método atribui valores às variáveis de *Agent State.* No caso de o
sistema possuir um *Radio*, 'state' é responsável por chamar o método
'getFirstMsg'.

10. **Radio2::Radio::getFirstMsg()**

Retorna a primeira mensagem "ativa" da lista 'messages', considerando
que esta armazena as mensagens de acordo com valores crescentes para a
seguinte variável 't':

t = 'timeStamp' + 'voiceTimeLength'.

A chamada a este método significa que a mensagem será removida da lista
'messages'. Portanto, a classe que chama o método torna-se responsável
por "tratar" a mensagem fornecida como retorno.
