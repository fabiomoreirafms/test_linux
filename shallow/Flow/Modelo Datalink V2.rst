**Datalink Flow**

1. **Player1::Player::runTimeCriticalRoutines(Phase 1)**

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
seus sistemas (classes *System*). Dessa forma, todas as classes
relacionadas a comunicações por datalink serão ativadas.

Cada *System*, ao receber uma chamada ao método
'runTimeCriticalRoutines', em função da fase corrente definida pela
classe *WorldModel*, irá chamar o método correspondente de sua
estrutura: 'dynamics', 'transmit', 'receive' e 'process'.

2. **Datalink1::Datalink::System::runTimeCriticalRoutines(Phase 1)**

Semelhantemente, o *Player* repassa a chamada de
'runTimeCriticalRoutines' para o *Datalink*, no momento que a fase 1 é
ativada.

Na fase 1, métodos relacionados à transmissão serão chamados, por meio
do método ‘transmit’.

3. **Datalink1::Datalink::transmit()**

O método 'transmit' é composto pelos seguintes procedimentos:

3.1 - Verifica se o *Datalink* está em modo silencioso ('silentMode').
Se estiver em modo silencioso ('silentMode' = TRUE), então nada será
realizado.

3.2 - Se não estiver no modo silencioso ('silentMode' = FALSE), então o
método continua.

3.3 - Se o *Datalink* estiver habilitado a transmitir mensagens de
status ('txStatusMsg' = TRUE) e o Player for um Fighter ou um AEW, então
uma mensagem de status é transmitida por meio do método
'transmitStatusMessage'.

3.4 - Se o *Datalink* estiver habilitado a transmitir mensagens de track
list ('txTrackListMsg' = TRUE), então, dependendo do tipo do *Player*
ownship, métodos diferentes serão utilizados na transmissão, da seguinte
forma:

+-----------------------------------+-----------------------------------+
| Tipo de *Player*                  | Método usado na transmissão de    |
|                                   | *Track List*                      |
+===================================+===================================+
| *Fighter*                         | 'transmitTrackListFighter'        |
+-----------------------------------+-----------------------------------+
| *AEW*                             | 'transmitTrackListAEW'            |
+-----------------------------------+-----------------------------------+
| *Air Surveillance Radar (ASR)*    | 'transmitTrackListGroundRadar'    |
+-----------------------------------+-----------------------------------+
| *Air Defense Radar (ADR)*         | 'transmitTrackListGroundRadar'    |
+-----------------------------------+-----------------------------------+
| *GBAD – Fire Unit (FU)*           | 'transmitTrackListGBAD_FU'        |
+-----------------------------------+-----------------------------------+
| *GBAD – C2*                       | 'transmitTrackListGBAD_C2'        |
+-----------------------------------+-----------------------------------+
| *C2 Center (C2C)*                 | 'transmitTrackListC2C'            |
+-----------------------------------+-----------------------------------+

A seguir, um pseudo-código do método 'transmit' é fornecido.

| Datalink::System::transmit()
| {
|     **if** (silentMode) **return**
|    
|     **if** (txStatusMsg)
|     {
|         // only Fighter and AEW can send Status Message
|         **if** ( ownship.classType == Fighter \|\| ownship.classType
  == AEW )
|             transmitStatusMessage()
|     }
|    
|     **if** (txTrackListMsg)
|     {
|         **if** ( ownship.classType == Fighter )
|             transmitTrackListFighter()
|            
|         **else** **if** ( ownship.classType == AEW )
|             transmitTrackListAEW()
|            
|         **else** **if** ( ownship.classType == GroundRadar )
|             transmitTrackListGroundRadar()
|            
|         **else** **if** ( ownship.classType == GBAD-FU )
|             transmitTrackListGBAD_FU()
|  
|         **else** **if** ( ownship.classType == GBAD-C2 )
|             transmitTrackListGBAD_C2()
|            
|         **else** **if** ( ownship.classType == C2C )
|             transmitTrackListC2C()
|     }
| }

Uma descrição detalhada dos métodos 'transmitStatusMessage',
'transmitTrackListFighter', 'transmitTrackListAEW',
'transmitTrackListGroundRadar', 'transmitTrackListGBAD_FU',
'transmitTrackListGBAD_C2', 'transmitTrackListC2C' é apresentada a
seguir.

a. **Datalink1::Datalink::transmitStatusMessage()**

a.1 - Uma lista com todos os players da simulação ('plist') é percorrida
para verificar qual a classe de cada *Player*.

a.1.1 - Se o *Player* é um *Fighter* ou um *AEW* e pertence à mesma
net/subnet que o ownship, o método 'sendMessage' é chamado passando o
status.

a.1.2 - Se o *Player* for um C2C e pertence à mesma net do ownship, o
método 'sendMessage'' é chamado passando o status.

| Datalink::transmitStatusMessage()
| {
|     // get list of all players (same 'side' as ownship)
|     plist = simulation.PlayersList
|    
|     **for** (each player in plist)
|     {
|         // send status message to fighters/aews, only if they belong
  to same net/subnet as my ownship
|         **if** ( player.classType == Fighter \|\| player.classType ==
  AEW )
|         {
|             **if** ( player.net == ownship.net && player.subnet ==
  ownship.subnet )
|                 sendMessage(Status)
|         }
|        
|         // send status message to C2C, only if it belongs to same net
  as my ownship
|         **else** **if** ( player.classType == C2C )
|         {
|             **if** ( player.net == ownship.net )
|                 sendMessage(Status)
|         }
|     }
| }

b. **Datalink1::Datalink::transmitTrackListFighter()**

b.1 - Uma lista com todos os players da simulação ('plist') é percorrida
para verificar qual a classe de cada *Player*.

b.1.1 - Se o *Player* é um *Fighter* e pertence à mesma
net/subnet/formation que o ownship, o método 'sendMessage' é chamado
passando a track list.

| Datalink::transmitTrackListFighter()
| {
|     // get list of all players (same 'side' as ownship)
|     plist = simulation.PlayersList
|    
|     **for** (each player in plist)
|     {
|         // From Fighter To Fighter - send track list message to
  fighters only if they belong to same net/subnet/formation as my
  ownship
|         **if** ( player.classType == Fighter )
|         {
|             **if** ( player.net == ownship.net && player.subnet ==
  ownship.subnet && player.formation == ownship.formation )
|                 sendMessage(TrackList)
|         }
|     }
| }

c. **Datalink1::Datalink::transmitTrackListAEW()**

c.1 - Uma lista com todos os players da simulação ('plist') é percorrida
para verificar qual a classe de cada *Player*.

c.1.1 - Se o *Player* é um *C2C* e pertence à mesma net que o ownship, o
método 'sendMessage' é chamado passando a track list.

| Datalink::transmitTrackListAEW()
| {
|     // get list of all players (same 'side' as ownship)
|     plist = simulation.PlayersList
|    
|     **for** (each player in plist)
|     {
|         // From AEW To C2C - send track list message to C2C only if
  they belong to same net as my ownship
|         **if** ( player.classType == C2C )
|         {
|             **if** ( player.net == ownship.net )
|                 sendMessage(TrackList)
|         }
|     }
| }

d. **Datalink1::Datalink::transmitTrackListGroundRadar()**

d.1 - Uma lista com todos os players da simulação ('plist') é percorrida
para verificar qual a classe de cada *Player*.

d.1.1 - Se o *Player* é um *C2C* e pertence à mesma net que o ownship, o
método 'sendMessage' é chamado passando a track list.

| Datalink::transmitTrackListGroundRadar()
| {
|     // get list of all players (same 'side' as ownship)
|     plist = simulation.PlayersList
|    
|     **for** (each player in plist)
|     {
|         // From GroundRadar to C2C - send track list message to C2C
  only if they belong to same net as my ownship
|         **if** ( player.classType == C2C )
|         {
|             **if** ( player.net == ownship.net )
|                 sendMessage(TrackList)
|         }
|     }
| }

e. **Datalink1::Datalink::transmitTrackListGBAD_FU()**

e.1 - Verificar se há um *GBAD*-*C2* associado ao ownship, player para o
qual o *GBAD*-*FU* está habilitado a transmitir mensagens de track list.

e.1.1 - Caso exista a associação, o método 'sendMessage' é chamado
passando a track list para o *GBAD*-*C2*.

e.1.2 - O método ‘sendMessage’ é chamado uma segunda vez passando também
as informações de engajamento dos alvos (‘TgtEngagementReport’).

| transmitTrackListGBAD_FU()
| {
|     // get GBAD-C2 player associated with my ownship (if it exists, as
  it's not mandatory)
|     player = ownship.getGBAD_C2()
|    
|     **if** (player != null)
|     {
|         // From GBAD-FU to GBAD-C2 - send track list message to
  GBAD-C2 (doesn't matter net and subnet set, it's assumed that datalink
  ok)
|         sendMessage(TrackList)
|        
|         // From GBAD-FU to GBAD-C2 - send target engagement report
  message to GBAD-C2 (doesn't matter net and subnet set, it's assumed
  that datalink ok)
|         sendMessage(TgtEngagementReport)
|     }
| }

f. **Datalink1::Datalink::transmitTrackListGBAD_C2()**

f.1 - Uma lista com todos os players *GBAD-FU* associados ao ownship
(‘fuList’) é percorrida.

f.1.1 - O método 'sendMessage' é chamado passando a track list.

| Datalink::transmitTrackListGBAD_C2()
| {
|     // get list of all fire units associated with my ownship
|     fuList = ownship.getListGBAD_FU()
|    
|     **for** (each fu in fuList)
|     {
|         // From GBAD-C2 to GBAD-FU - send track list message to
  GABD-FU (doesn't matter net and subnet set, it's assumed that datalink
  ok)
|         sendMessage(TrackList)
|     }
| }

g. **Datalink1::Datalink::transmitTrackListC2C()**

g.1 - Uma lista com todos os players da simulação ('plist') é percorrida
para verificar qual a classe de cada *Player*.

g.1.1 - Se o ground link está ativo ('hasGroundLink' = TRUE), o *Player*
é um *Fighter* e pertence à mesma net que o ownship, o método
'sendMessage' é chamado passando a track list.

| Datalink::transmitTrackListC2C()
| {
|     // get list of all players (same 'side' as ownship)
|     plist = simulation.PlayersList
|    
|     **for** (each player in plist)
|     {
|         // From C2C to Fighter - send track list message to fighters,
  only if they belong to same net as my ownship and if 'hasGroundLink'
  is true
|         **if** ( player.classType == Fighter && hasGroundLink )
|         {
|             **if** ( player.net == ownship.net )
|                 sendMessage(TrackList)
|         }
|     }
| }

4. **Datalink1::Datalink::sendMessage()**

Chama o método 'triggerEvent(DATALINK_MESSAGE)' do *Player*
destinatário, desde que este esteja a uma distância inferior ao valor do
atributo 'maxRange'.

Se estiver a uma distância superior a 'maxRange', tenta encontrar alguém
da sua net/subnet que possua a função 'relay' ativada e esteja a uma
distância inferior a 'maxRange' tanto do remetente quanto do
destinatário.

5. **Player2::Player::triggerEvent(DATALINK_MESSAGE)**

Aciona o método 'handleDatalinkMessageEvent' do *Player*, que inicia o
processo de recepção da mensagem.

6. **Player2::Player::handleDatalinkMessageEvent()**

Repassa a mensagem recebida para o *Datalink* (caso possua o equipamento
definido em sua estrutura), acionando o método
'handleDatalinkMessageEvent' do mesmo.

7. **Datalink2::Datalink::handleDatalinkMessageEvent()**

Verifica se o destinatário da mensagem coincide com o ownship. Se
coincidir, inclui a mensagem recebida na lista 'inMsgQueue'.

8. **Player2::Player::runTimeCriticalRoutines(Phase 2)**

Similar ao descrito no item 1, entretanto, neste caso, a phase 2
'receive' do *Player* é chamada.

9. **Datalink2::Datalink::runTimeCriticalRoutines(Phase 2)**

Semelhantemente, o *Player* repassa a chamada de
'runTimeCriticalRoutines' para o *Datalink*, no momento que a fase 2 é
ativada.

Na fase 2, os métodos relacionados à recepção são tratados a partir do
método 'receive'. Assim, o método 'receive' do *Datalink* é acionado.

10. **Datalink2::Datalink::receive()**

Para cada mensagem na fila de mensagens ('inMsgQueue'), verifica-se qual
o seu tipo e chama o método correspondente de recepção.

| Datalink::System::receive()
| {
|     **for** (each msg in inMsgQueue)
|     {
|         **if** (msg.type == AirVehicleStatus && rxStatusMsg)
|             receiveAirVehicleStatusMessage()
|            
|         **else** **if** (msg.type == AirBaseStatus && rxStatusMsg)
|             receiveAirBaseStatusMessage()
|            
|         **else** **if** (msg.type == TaskOrder)
|             receiveTaskOrderMessage()
|            
|         **else** **if** (msg.type == LandingReport)
|             receiveLandingReportMessage()
|            
|         **else** **if** (msg.type == TrackList && rxTrackListMsg)
|             receiveTrackListMessage()
|            
|         **else** **if** (msg.type == TgtEngagementReport)
|             receiveTgtEngReportMessage()
|            
|         **else** **if** (msg.type == TgtEngagementOrder)
|             receiveTgtEngOrderMessage()
|     }
| }

Uma descrição detalhada dos métodos 'receiveAirVehicleStatusMessage',
'receiveAirBaseStatusMessage', 'receiveTaskOrderMessage',
'receiveLandingReportMessage', 'receiveTgtEngReportMessage',
'receiveTgtEngOrderMessage' e 'receiveTrackListMessage' é apresentada a
seguir.

a. **Datalink2::Datalink::receiveAirVehicleStatusMessage()**

Se a mensagem for do tipo status e proveniente de *AirVehicle*, é feita
a verificação da classe que o ownship pertence, pois somente
*AirVehicle* (*Fighter* e AEW) e *C2C* podem recebê-la. Se esse for o
caso, é chamado o método 'triggerEvent' da plataforma de destino com o
parâmetro 'AV_STATUS'.

| Datalink::receiveAirVehicleStatusMessage()
| {
|     **if** (ownship.classType == AirVehicle)
|         airVehicle.triggerEvent(AV_STATUS)
|        
|     **else** **if** (ownship.classType == C2C)
|         c2c.triggerEvent(AV_STATUS)
| }

b. **Datalink2::Datalink::receiveAirBaseStatusMessage()**

Se a mensagem for do tipo status e proveniente de *AirBase*, é feita a
verificação da classe que o ownship pertence, pois somente *C2C* pode
recebê-la. Se esse for o caso, é chamado o método 'triggerEvent' da
plataforma de destino com o parâmetro 'AB_STATUS'.

| Datalink::receiveAirBaseStatusMessage()
| {
|     **if** (ownship.classType == C2C)
|         c2c.triggerEvent(AB_STATUS)
| }

c. **Datalink2::receiveTaskOrderMessage()**

Se a mensagem for do tipo task order, é feita a verificação da classe
que o ownship pertence, pois somente *AirBase* pode recebê-la. Se esse
for o caso, é chamado o método 'triggerEvent' da plataforma de destino
com o parâmetro 'TASK_ORDER'.

| Datalink::receiveTaskOrderMessage()
| {
|     **if** (ownship.classType == AirBase)
|         airBase.triggerEvent(TASK_ORDER)
| }

d. **Datalink2::Datalink::receiveLandingReportMessage()**

Se a mensagem for do tipo landing report, é feita a verificação da
classe que o ownship pertence, pois somente *AirBase* pode recebê-la. Se
esse for o caso, é chamado o método 'triggerEvent' da plataforma de
destino com o parâmetro 'LANDING_REPORT'.

| Datalink::receiveLandingReportMessage()
| {
|     **if** (ownship.classType == AirBase)
|         airBase.triggerEvent(LANDING_REPORT)
| }

e. **Datalink2::receiveTgtEngReportMessage()**

Se a mensagem for do tipo target engagement report, é feita a
verificação da classe que o ownship pertence, pois somente *GBAD-C2*
pode recebê-la. Se esse for o caso, é chamado o método 'triggerEvent' da
plataforma de destino com o parâmetro 'TGT_ENG_REPORT'.

| Datalink::receiveTgtEngReportMessage()
| {
|     **if** (ownship.classType == GBAD-C2)
|         gbad_c2.triggerEvent(TGT_ENG_REPORT)
| }

f. **Datalink2::receiveTgtEngOrderMessage()**

Se a mensagem for do tipo target engagement order, é feita a verificação
da classe que o ownship pertence, pois somente *GBAD-FU* pode recebê-la.
Se esse for o caso, é chamado o método 'triggerEvent' da plataforma de
destino com o parâmetro 'TGT_ENG_ORDER'.

| Datalink::receiveTgtEngReportOrder()
| {
|     **if** (ownship.classType == GBAD-FU)
|         gbad_fu.triggerEvent(TGT_ENG_ORDER)
| }

g. **Datalink2::Datalink::receiveTrackListMessage()**

Se a mensagem for do tipo track list, é feita a verificação da classe
que o ownship pertence, pois somente *Fighter*, *C2C*, *GBAD-C2* e
*GBAD-FU* podem recebê-la. Se esse for o caso, gerar *Emission* a partir
da mensagem recebida e enviar novo reporte para o *TrackManager* do
ownship\ *.*

| receiveTrackListMessage()
| {
|     **if** ( ownship.classType == Fighter \|\| ownship.classType ==
  C2C \|\| ownship.classType == GBAD-C2 \|\| ownship.classType ==
  GBAD-FU )
|         // gerar 'Emission' a partir da mensagem recebida 
|         // enviar novo reporte para o track manager do ownship
| }
