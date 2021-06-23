**Dynamics Flow**

1. **Player::runTimeCriticalRoutines**

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

2. **Player::updateDynamics()**

O método realiza as seguintes ações:

   Chamada ao método 'dynamics' da classe Dynamics Model (caso exista um
   modelo dinâmico associado ao player); a chamada a este método faz com
   que os atributos 'eulerAngles', 'angularVel', 'velocity' e 'accel'
   sejam atualizados;

   Cálculo da nova posição do player e atualização dos atributos
   'latitude', 'longitude', 'altitude' e 'position';

   Verificação de colisão do player com o solo; caso positivo, muda o
   valor do atributo 'mode' para "crashed".

3. **Aircraft Dynamics Model::dynamics()**

..

   Calcula o fluxo de combustível e atualiza o valor do combustível
   remanescente e do peso total;

   Calcula os novos valores dos atributos 'phi', 'theta', 'psi' e 'u',
   realizando integração a partir dos atributos "dot";

   Chama os métodos 'setEulerAngles', 'setAngularVelocities' e
   'setLinearVelocities' do Player.

Ver detalhes dos cálculos no método de mesmo nome na classe Laero do
framework MIXR.

4. **Player::setEulerAngles()**

Define valores para os atributos da classe *Agent Action*.

É onde os métodos principais de inteligência artificial devem ser
implementados, a fim de determinar quais serão as ações (*Agent Action*)
que o *Agent* deve tomar a partir de seu estado (*Agent State*).

Método abstrato; deve ser implementado pelas classes herdeiras.

5. **Player::setAngularVelocities()**

Método chamado por outras classes (ex: *Dynamics Model*) para definir o
valor do atributo 'angularVel'.

6. **Player::setLinearVelocities()**

Método chamado por outras classes (ex: *Dynamics Model*) para definir o
valor do atributo 'velocity'.

7. **Agent::runTimeCriticalRoutines()**

Semelhantemente, o *Player* repassa a chamada de
'runTimeCriticalRoutines' para o *Agent*, no momento que a fase 3 é
ativada.

8. **Aircraft Dynamics Model::flyAltitudeSpeed()**

- Calcula o climb rate;

- Chama setCommandedAltitude da classe base;

- Calcula a aceleração longitudinal;

- Chama setCommandedVelocity da classe base.

Consulte a documentação específica do modelo para maiores detalhes sobre
o método de cálculo.

9. **Aircraft Dynamics Model:: setCommandedAltitude()**

Esta classe calcula o valor do atributo 'thetaDot'. Ver detalhes dos
cálculos no método de mesmo nome na classe Laero do framework MIXR.

Quem chama este método é a classe Aircraft Dynamics Model, a partir de
uma chamada ao seu método flyAltitudeSpeed.

10. **Aircraft Dynamics Model:: setCommandedVelocity()**

Esta classe calcula o valor do atributo 'uDot'. Ver detalhes dos
cálculos no método de mesmo nome na classe Laero do framework MIXR.

Quem chama este método é a classe *Aircraft Dynamics Model*, a partir de
uma chamada ao seu método 'flyAltitudeSpeed'.

11. **Aircraft Dynamics Model::flyHeading()**

- Calcula o g disponível e define o g comandado (limitando o requerido,
se for o caso);

- Calcula o 'rate of turn' resultante a partir do g comandado;

- Calcula o bank angle em função do g comandado e verifica o limite.

- Chama o método 'setCommandedHeading' da classe base.

Consulte a documentação específica do modelo para maiores detalhes sobre
o método de cálculo.

12. **Aircraft Dynamics Model::setCmdHeading()**

Este método calcula o novo valor do atributo 'phiDot'.

O novo valor do atributo 'psiDot' é definido como uma função da
diferença entre as proas comandadas e atual, e do valor do parâmetro
'rateChange'.

Ver detalhes dos cálculos no método de mesmo nome na classe Laero do
framework MIXR.

Quem chama este método é a classe *Aircraft Dynamics Model*, a partir de
uma chamada ao seu método 'flyHeading'.
