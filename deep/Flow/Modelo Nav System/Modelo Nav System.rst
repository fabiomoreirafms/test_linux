**Modelo Nav System**

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

Como consequência, cada *Player* irá chamar o mesmo método de cada um de
seus sistemas (classes *System*). Cada *System*, ao receber uma chamada
ao método 'runTimeCriticalRoutines', em função da fase corrente definida
pela classe *WorldModel*, irá chamar o método correspondente de sua
estrutura: 'dynamics', 'transmit', 'receive' e 'process'.

2. **NavSystem::runTimeCriticalRoutines(Phase 3)**

Semelhantemente, o Player repassa a chamada de 'runTimeCriticalRoutines'
para o *NavSystem*, no momento que a fase 3 é ativada.

Na fase 3, métodos relacionados ao processamento serão chamados, por
meio do método ‘process’.

3. **NavSystem::process()**

O método 'process' realiza as seguintes ações:

a - atualiza o valor da variável 'utc', que corresponde a hora (UTC)
corrente no Sistema de Navegação.

b - chama o método 'runTimeCriticalRoutines' da classe 'Route'.

c - realiza chamadas aos seguintes métodos:

c.1 - updateSysPosition.

c.2 - updateSysAttitude.

c.3 - updateSysVelocity.

c.4 - updateNavSteering.

4. **Route::runTimeCriticalRoutines()**

Chama os seguintes métodos da classe Route:

- 'computeSteerpointData';

- 'autoSequencer'.

5. **Route::computeSteerpointData()**

Chama o método 'compute' de cada steerpoint presente na lista
'steerpoints', passando os parâmetros 'nav' e 'from' (steerpoint
anterior na rota).

6. **Steerpoint::compute()**

Calcula os valores dos atributos de estado do steerpoint, a partir de
informações do *NavSystem* e de um steerpoint anterior ('from'). Os
atributos de estado do steerpoint são descritos a seguir.

- toDistance: distância entre a posição atual da plataforma (obtida do
Nav System) e o steerpoint, em modo "direct to".

- toTTG: tempo para alcançar o steerpoint, em modo "direct to",
considerando a posição atual da plataforma (obtida de Nav System).

- toTrueBearing: proa (direção) entre a posição atual da plataforma
(obtida do Nav System) e o steerpoint, em modo "direct to".

- legDistance: Comprimento da 'leg' definida entre um steerpoint de
origem ('from') e o próprio steerpoint.

- legTTG: Tempo gasto para percorrer a 'leg' definida entre um
steerpoint de origem ('from') e o próprio steerpoint.

- legTrueCourse: proa (direção) da 'leg' definida entre um steerpoint de
origem ('from') e o próprio steerpoint.

- legTrackError: erro lateral (distância) entre o curso pretendido e o
curso atual.

- enrouteDistance: distância acumulada desde o início da rota até o
steerpoint.

- enrouteETE: Tempo estimado desde o início da rota até o steerpoint.
ETE - Estimated Time Enroute.

- enrouteETA: Hora estimada (UTC) de sobrevoo do steerpoint. ETA -
Estimated Time of Arrival.

- stptEarlyLateTime: Diferença entre enrouteETA e stptPTA.

7. **Route::autoSequencer()**

Se o atributo 'autoSequence' for true, se a distância para o steerpoint
'directTo' for menor que a definida no atributo 'autoSeqDis', e se o
steerpoint 'directTo' já tiver sido sobrevoado, então, o próximo
steerpoint da rota é definido como 'directTo'.

8. **NavSystem::updateSysPosition()**

Atualiza as seguintes variáveis do sistema NavSystem com valores
oriundos do Player:

- latitude: latitude corrente no Sistema de Navegação.

- longitude: longitude corrente no Sistema de Navegação.

- altitude: altitude corrente no Sistema de Navegação.

9. **NavSystem::updateSysAttitude()**

Atualiza as seguintes variáveis do sistema NavSystem com valores
oriundos do Player:

- heading: proa corrente no Sistema de Navegação.

- pitch: arfagem (atitude) corrente no Sistema de Navegação.

- roll: inclinação (bank) corrente no Sistema de Navegação.

10. **NavSystem::updateSysVelocity()**

Atualiza as seguintes variáveis do sistema *NavSystem* com valores
oriundos do *Player*:

- gndSpeed: ground speed corrente no Sistema de Navegação.

- tas: True Air Speed (TAS) corrente no Sistema de Navegação.

- gndTrack: direção do deslocamento da plataforma medida no solo.

11. **NavSystem::updateNavSteering()**

Atualiza as seguintes variáveis do sistema *NavSystem* com valores
oriundos de 'route':

- toTrueBearing: proa para o steerpoint 'directTo' selecionado.

- toTrueCourse: curso para o steerpoint 'directTo' selecionado.

- toDistance: distância para o steerpoint 'directTo' selecionado.

- toTrackError: erro lateral (distância) entre o curso pretendido e o
curso atual.

- toTTG: tempo restante para alcançar o steerpoint 'directTo'
selecionado.

- toETA: hora prevista (UTC) de sobrevoo do steerpoint 'directTo'
selecionado.

- toEarlyLateTime: diferença entre enrouteETA e stptPTA.
