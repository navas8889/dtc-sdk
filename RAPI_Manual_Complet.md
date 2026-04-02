# Manuel complet R|API+

Version reconstruite à partir d’un export Doxygen HTML.

## Table des matières
- [Manuel R|API+ reconstruit depuis la documentation Doxygen HTML](#manuel-rapi-reconstruit-depuis-la-documentation-doxygen-html)
- [Paramètres de requête, d’initialisation et cycle de vie](#parametres-de-requête-dinitialisation-et-cycle-de-vie)
- [Données de marché, référentiel et historique](#donnees-de-marche-referentiel-et-historique)
- [Trading, comptes, PnL et état](#trading-comptes-pnl-et-etat)
- [Administration, recherche, reprise et rejeu](#administration-recherche-reprise-et-rejeu)
- [Annexes : constantes et index](#annexes--constantes-et-index)

---

# Manuel R|API+ reconstruit depuis la documentation Doxygen HTML

## Table des matières
- [1. Vue d’ensemble](#1-vue-densemble)
- [2. Architecture logique](#2-architecture-logique)
- [3. Composants critiques](#3-composants-critiques)
- [4. Relations entre composants](#4-relations-entre-composants)
- [5. Espaces de noms, fichiers et conventions](#5-espaces-de-noms-fichiers-et-conventions)

## 1. Vue d’ensemble

Cette version est une reconstruction éditée d’une documentation Doxygen HTML. Le bruit de navigation a été supprimé, les éléments redondants ont été fusionnés, et l’ordre a été réorganisé pour être exploitable comme un manuel technique.

### Objectif de l’API

L’API Rithmic (`R|API`) est conçue pour des usages de trading à faible latence et à fort débit. Elle expose un modèle unifié pour accéder aux données de marché, soumettre des ordres, récupérer l’état applicatif après redémarrage et recevoir des alertes d’infrastructure.

Les objectifs mis en avant dans la documentation sont les suivants :

- **latence minimale** dans le transit des messages marché et trading ;
- **fort throughput** côté client, en particulier pour les flux marché ;
- **normalisation** des protocoles et formats propres à chaque place ou passerelle ;
- **masquage des mécanismes de failover et de recovery** au niveau de l’infrastructure ;
- **simplification du code client** grâce à des primitives de récupération d’état, de consultation des ordres ouverts, des positions et d’une partie de l’historique ;
- **cohérence d’interface** via des conventions homogènes de types, noms et structures.

Les domaines fonctionnels principaux sont :

- données de marché : trades, quotes, order book, références instrument ;
- placement d’ordres : limit, market, stop limit, stop market ;
- reprise d’état : ordres ouverts, positions, audit trail, historiques ;
- alertes et messages de contrôle : état des connexions, login, trading enable/disable, shutdown, etc.

### Lecture opérationnelle

L’API est structurée autour de quatre axes :

1. **Initialisation moteur** avec `REngineParams` puis création de `REngine`.
2. **Ouverture de session** avec `LoginParams` et enregistrement des callbacks `RCallbacks`.
3. **Consommation événementielle** via les nombreuses structures `*Info`, `*Report` et `*ReplayInfo` envoyées de façon asynchrone.
4. **Actions client** via `REngine` et les structures `*Params` pour requêtes de marché, historique et ordres.

## 2. Architecture logique

### Flux nominal

```text
Application cliente
  -> REngineParams
  -> REngine
  -> LoginParams
  -> REngine::login(...)
  -> infrastructure Rithmic
  -> RCallbacks / AdmCallbacks
  -> structures *Info / *Report / *ReplayInfo
```

### Macro-composants

- **Moteur** : `REngine`.
- **Configuration moteur** : `REngineParams`.
- **Configuration de session** : `LoginParams`.
- **Callbacks métier** : `RCallbacks`.
- **Callbacks administratifs** : `AdmCallbacks`.
- **Timers** : `OmneTimerCB` + méthodes timer de `REngine`.
- **Structures de données** : classes `*Info`, `*Report`, `*Params`, `*ReplayInfo`.

## 3. Composants critiques

### `REngine`

- **Rôle** : This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.
- **Include** : `#include <RApiPlus.h>`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
int getAppVersion (tsNCharcb *pAppVersion, int *aiCode)
int getAuxRefData (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int getCallbacks (RCallbacks **apCallbacks, int *aiCode)
int getContext (void **apContext, int *aiCode)
int getEasyToBorrowList (void *pContext, int *aiCode)
int getEnvironment (tsNCharcb *pKey, void *pContext, int *aiCode)
int getEquityOptionStrategyList (tsNCharcb *pExchange, tsNCharcb *pUnderlying, tsNCharcb *pStrategyType, tsNCharcb *pExpiration, void *pContext, int *aiCode)
int getIhCnnctPt (tsNCharcb *pIhCnnctPt, int *aiCode)
int getInstrumentByUnderlying (tsNCharcb *pUnderlying, tsNCharcb *pExchange, tsNCharcb *pExpiration, void *pContext, int *aiCode)
int getMdCnnctPt (tsNCharcb *pMdCnnctPt, int *aiCode)
int getOmne (void **apOmne, int *aiCode)
int getOptionList (tsNCharcb *pExchange, tsNCharcb *pProductCode, tsNCharcb *pExpirationCCYYMM, void *pContext, int *aiCode)
int getOrderContext (tsNCharcb *pOrderNum, void **apContext, int *aiCode)
int getPassword (tsNCharcb *pPassword, int *aiCode)
int getPendingInputSize (int iConnId, unsigned long int *aulPendingBytes, int *aiCode)
int getPnlCnnctPt (tsNCharcb *pPnlCnnctPt, int *aiCode)
int getPriceIncrInfo (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int getProductRmsInfo (AccountInfo *pAccount, int *aiCode)
int getRefData (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int getStrategyInfo (tsNCharcb *pExchange, tsNCharcb *pTicker, void *pContext, int *aiCode)
int getStrategyList (tsNCharcb *pExchange, tsNCharcb *pProductCode, tsNCharcb *pStrategyType, tsNCharcb *pExpirationCCYYMM, void *pContext, int *aiCode)
int getTsCnnctPt (tsNCharcb *pTsCnnctPt, int *aiCode)
int getUser (tsNCharcb *pUser, int *aiCode)
int getUserProfile (int iConnId, void *pContext, int *aiCode)
int getVolumeAtPrice (tsNCharcb *pExchange, tsNCharcb *pTicker, void *pContext, int *aiCode)
int isThereAnAggregator (int *aiCode)
int linkOrders (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, int iArrayLen, int *aiCode)
int listAgreements (bool bAccepted, void *pContext, int *aiCode)
int listAssignedUsers (AccountInfo *pAccount, void *pContext, int *aiCode)
int listBinaryContracts (tsNCharcb *pExchange, tsNCharcb *pProductCode, void *pContext, int *aiCode)
int listEnvironments (void *pContext, int *aiCode)
int listExchanges (void *pContext, int *aiCode)
int listIbs (void *pContext, int *aiCode)
int listOrderHistoryDates (void *pContext, int *aiCode)
int listTradeRoutes (void *pContext, int *aiCode)
int listUsers (tsNCharcb *pIbId, tsNCharcb *pUserType, void *pContext, int *aiCode)
int lock (int *aiCode)
int login (LoginParams *pParams, int *aiCode)
int loginRepository (tsNCharcb *pEnvKey, tsNCharcb *pUser, tsNCharcb *pPassword, tsNCharcb *pCnnctPt, RCallbacks *pCallbacks, int *aiCode)
int logout (int *aiCode)
int logoutRepository (int *aiCode)
int modifyBracketTier (AccountInfo *pAccount, tsNCharcb *pOrderNum, bool bTarget, int iOldTickOffset, int iNewTickOffset, void *pContext, int *aiCode)
int modifyOrder (ModifyLimitOrderParams *pParams, int *aiCode)
int modifyOrder (ModifyOrderParams *pParams, int *aiCode)
int modifyOrder (ModifyStopLimitOrderParams *pParams, int *aiCode)
int modifyOrder (ModifyStopMarketOrderParams *pParams, int *aiCode)
int modifyOrderList (ModifyOrderParams *pParams, int iArrayLen, int *aiCode)
int modifyOrderRefData (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pUserMsg, tsNCharcb *pUserTag, int *aiCode)
int rebuildBook (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int rebuildDboBook (tsNCharcb *pExchange, tsNCharcb *pTicker, bool bPrice, double dPrice, void *pContext, int *aiCode)
int removeTimer (tsNCharcb *const pTimerName, int *aiCode)
REngine (REngineParams *pParams)
int replayAllOrders (AccountInfo *pAccount, int iStartSsboe, int iEndSsboe, int *aiCode)
int replayBars (ReplayBarParams *pParams, int *aiCode)
int replayBrackets (AccountInfo *pAccount, int *aiCode)
int replayExecutions (AccountInfo *pAccount, int iStartSsboe, int iEndSsboe, int *aiCode)
int replayHistoricalOrders (AccountInfo *pAccount, tsNCharcb *pDate, int *aiCode)
int replayOpenOrders (AccountInfo *pAccount, int *aiCode)
int replayPnl (AccountInfo *pAccount, int *aiCode)
int replayQuotes (AccountInfo *pAccount, void *pContext, int *aiCode)
int replaySingleHistoricalOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pDate, void *pContext, int *aiCode)
int replaySingleOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, void *pContext, int *aiCode)
int replayTrades (tsNCharcb *pExchange, tsNCharcb *pTicker, int iStartSsboe, int iEndSsboe, int *aiCode)
int resumeInput (int *aiCode)
int searchInstrument (tsNCharcb *pExchange, SearchTerm *asTerms, int iNumTerms, void *pContext, int *aiCode)
int sendBracketOrder (OrderParams *pEntry, BracketParams *pBracketParams, int *aiCode)
int sendBracketOrder (OrderParams *pEntry, tsNCharcb *pBracketType, tsNCharcb *pOrderOperationType, bool bTrailingStop, int iTrailByPriceId, int iTargetArrayLen, int *pTargetQty, int *pTargetTicks, int iStopArrayLen, int *pStopQty, int *pStopTicks, int *aiCode)
int sendOcaList (tsNCharcb *pOcaType, OrderParams *pParamsArray, int iArrayLen, int *aiCode)
int sendOrder (LimitOrderParams *pParams, int *aiCode)
int sendOrder (MarketOrderParams *pParams, int *aiCode)
int sendOrder (StopLimitOrderParams *pParams, int *aiCode)
int sendOrder (StopMarketOrderParams *pParams, int *aiCode)
int sendOrderList (OrderParams *pParamsArray, int iArrayLen, int *aiCode)
int setContext (void *pContext, int *aiCode)
int setEnvironmentVariable (tsNCharcb *pKey, tsNCharcb *pVariable, tsNCharcb *pValue, int *aiCode)
int setOrderContext (tsNCharcb *pOrderNum, void *pContext, int *aiCode)
int startTimer (tsNCharcb *const pTimerName, int *aiCode)
int stopTimer (tsNCharcb *const pTimerName, int *aiCode)
int submitQuoteList (QuoteParams *pParams, int iArrayLen, int *aiCode)
int subscribe (tsNCharcb *pExchange, tsNCharcb *pTicker, int iFlags, int *aiCode)
int subscribeAutoLiquidate (AccountInfo *pAccount, int *aiCode)
int subscribeBar (BarParams *pParams, int *aiCode)
int subscribeBracket (AccountInfo *pAccount, int *aiCode)
int subscribeByUnderlying (tsNCharcb *pUnderlying, tsNCharcb *pExchange, tsNCharcb *pExpiration, int iFlags, void *pContext, int *aiCode)
int subscribeDbo (tsNCharcb *pExchange, tsNCharcb *pTicker, bool bPrice, double dPrice, void *pContext, int *aiCode)
int subscribeEasyToBorrow (void *pContext, int *aiCode)
int subscribeOrder (AccountInfo *pAccount, int *aiCode)
int subscribePnl (AccountInfo *pAccount, int *aiCode)
int subscribeTradeRoute (tsNCharcb *pFcmId, tsNCharcb *pIbId, void *pContext, int *aiCode)
int subscribeUser (tsNCharcb *pIbId, void *pContext, int *aiCode)
int suspendInput (int *aiCode)
int tryLock (int *aiCode)
int unlock (int *aiCode)
int unsetEnvironmentVariable (tsNCharcb *pKey, tsNCharcb *pVariable, int *aiCode)
int unsubscribe (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int unsubscribeAutoLiquidate (AccountInfo *pAccount, int *aiCode)
int unsubscribeBar (BarParams *pParams, int *aiCode)
int unsubscribeBracket (AccountInfo *pAccount, int *aiCode)
int unsubscribeByUnderlying (tsNCharcb *pUnderlying, tsNCharcb *pExchange, tsNCharcb *pExpiration, int *aiCode)
int unsubscribeDbo (tsNCharcb *pExchange, tsNCharcb *pTicker, bool bPrice, double dPrice, void *pContext, int *aiCode)
int unsubscribeEasyToBorrow (int *aiCode)
int unsubscribeOrder (AccountInfo *pAccount, int *aiCode)
int unsubscribePnl (AccountInfo *pAccount, int *aiCode)
int unsubscribeTradeRoute (tsNCharcb *pFcmId, tsNCharcb *pIbId, int *aiCode)
int unsubscribeUser (tsNCharcb *pIbId, int *aiCode)
virtual ~REngine ()
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `RCallbacks`

- **Rôle** : This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.
- **Include** : `#include <RApiPlus.h>`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int BestBidAskQuote (BidInfo *pBid, AskInfo *pAsk, void *pContext, int *aiCode)
virtual int BestBidQuote (BidInfo *pInfo, void *pContext, int *aiCode)
virtual int BidQuote (BidInfo *pInfo, void *pContext, int *aiCode)
virtual int BinaryContractList (BinaryContractListInfo *pInfo, void *pContext, int *aiCode)
virtual int BracketReplay (BracketReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BracketTierModify (BracketTierModifyInfo *pInfo, void *pContext, int *aiCode)
virtual int BracketUpdate (BracketInfo *pInfo, void *pContext, int *aiCode)
virtual int BustReport (OrderBustReport *pReport, void *pContext, int *aiCode)
virtual int CancelReport (OrderCancelReport *pReport, void *pContext, int *aiCode)
virtual int CloseMidPrice (CloseMidPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int ClosePrice (ClosePriceInfo *pInfo, void *pContext, int *aiCode)
virtual int ClosingIndicator (ClosingIndicatorInfo *pInfo, void *pContext, int *aiCode)
virtual int Dbo (DboInfo *pInfo, void *pContext, int *aiCode)
virtual int DboBookRebuild (DboBookRebuildInfo *pInfo, void *pContext, int *aiCode)
virtual int EasyToBorrow (EasyToBorrowInfo *pInfo, void *pContext, int *aiCode)
virtual int EasyToBorrowList (EasyToBorrowListInfo *pInfo, void *pContext, int *aiCode)
virtual int EndQuote (EndQuoteInfo *pInfo, void *pContext, int *aiCode)
virtual int EquityOptionStrategyList (EquityOptionStrategyListInfo *pInfo, void *pContext, int *aiCode)
virtual int ExchangeList (ExchangeListInfo *pInfo, void *pContext, int *aiCode)
virtual int ExecutionReplay (ExecutionReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int FailureReport (OrderFailureReport *pReport, void *pContext, int *aiCode)
virtual int FillReport (OrderFillReport *pReport, void *pContext, int *aiCode)
virtual int HighBidPrice (HighBidPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int HighPrice (HighPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int HighPriceLimit (HighPriceLimitInfo *pInfo, void *pContext, int *aiCode)
virtual int IbList (IbListInfo *pInfo, void *pContext, int *aiCode)
virtual int InstrumentByUnderlying (InstrumentByUnderlyingInfo *pInfo, void *pContext, int *aiCode)
virtual int InstrumentSearch (InstrumentSearchInfo *pInfo, void *pContext, int *aiCode)
virtual int LimitOrderBook (LimitOrderBookInfo *pInfo, void *pContext, int *aiCode)
virtual int LineUpdate (LineInfo *pInfo, void *pContext, int *aiCode)
virtual int LowAskPrice (LowAskPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int LowPrice (LowPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int LowPriceLimit (LowPriceLimitInfo *pInfo, void *pContext, int *aiCode)
virtual int MarketMode (MarketModeInfo *pInfo, void *pContext, int *aiCode)
virtual int MidPrice (MidPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int ModifyReport (OrderModifyReport *pReport, void *pContext, int *aiCode)
virtual int NotCancelledReport (OrderNotCancelledReport *pReport, void *pContext, int *aiCode)
virtual int NotModifiedReport (OrderNotModifiedReport *pReport, void *pContext, int *aiCode)
virtual int OpeningIndicator (OpeningIndicatorInfo *pInfo, void *pContext, int *aiCode)
virtual int OpenInterest (OpenInterestInfo *pInfo, void *pContext, int *aiCode)
virtual int OpenOrderReplay (OrderReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int OpenPrice (OpenPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int OptionList (OptionListInfo *pInfo, void *pContext, int *aiCode)
virtual int OrderHistoryDates (OrderHistoryDatesInfo *pInfo, void *pContext, int *aiCode)
virtual int OrderReplay (OrderReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int OtherReport (OrderReport *pReport, void *pContext, int *aiCode)
virtual int PasswordChange (PasswordChangeInfo *pInfo, void *pContext, int *aiCode)
virtual int PnlReplay (PnlReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int PnlUpdate (PnlInfo *pInfo, void *pContext, int *aiCode)
virtual int PositionExit (PositionExitInfo *pInfo, void *pContext, int *aiCode)
virtual int PriceIncrUpdate (PriceIncrInfo *pInfo, void *pContext, int *aiCode)
virtual int ProductRmsList (ProductRmsListInfo *pInfo, void *pContext, int *aiCode)
virtual int ProjectedSettlementPrice (ProjectedSettlementPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int Quote (QuoteInfo *pInfo, void *pContext, int *aiCode)
virtual int Quote (QuoteReport *pReport, void *pContext, int *aiCode)
virtual int QuoteReplay (QuoteReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int RefData (RefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int RejectReport (OrderRejectReport *pReport, void *pContext, int *aiCode)
virtual int SettlementPrice (SettlementPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int SingleOrderReplay (SingleOrderReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int SodUpdate (SodReport *pReport, void *pContext, int *aiCode)
virtual int StatusReport (OrderStatusReport *pReport, void *pContext, int *aiCode)
virtual int Strategy (StrategyInfo *pInfo, void *pContext, int *aiCode)
virtual int StrategyList (StrategyListInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeCondition (TradeInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeCorrectReport (OrderTradeCorrectReport *pReport, void *pContext, int *aiCode)
virtual int TradePrint (TradeInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeReplay (TradeReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeRoute (TradeRouteInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeRouteList (TradeRouteListInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeVolume (TradeVolumeInfo *pInfo, void *pContext, int *aiCode)
virtual int TriggerPulledReport (OrderTriggerPulledReport *pReport, void *pContext, int *aiCode)
virtual int TriggerReport (OrderTriggerReport *pReport, void *pContext, int *aiCode)
virtual int User (UserInfo *pInfo, void *pContext, int *aiCode)
virtual int UserDefinedSpreadCreate (UserDefinedSpreadCreateInfo *pInfo, void *pContext, int *aiCode)
virtual int UserList (UserListInfo *pInfo, void *pContext, int *aiCode)
virtual int UserProfile (UserProfileInfo *pInfo, void *pContext, int *aiCode)
virtual int VolumeAtPrice (VolumeAtPriceInfo *pInfo, void *pContext, int *aiCode)
virtual ~RCallbacks ()
```

### `AdmCallbacks`

- **Rôle** : This abstract class contains all the admin-related callbacks for RApi . A subclass of AdmCallbacks can be defined, instantiated and passed via REngineParams::pCallbacks when logging into the infrastructure. While the REngine instance is alive, the AdmCallbacks instance referenced by REngineParams::pCallbacks must not be destroyed.
- **Include** : `#include <RApiPlus.h>`

**Public Member Functions**
```cpp
AdmCallbacks ()
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)=0
virtual int Environment (EnvironmentInfo *pInfo, void *pContext, int *aiCode)
virtual int EnvironmentList (EnvironmentListInfo *pInfo, void *pContext, int *aiCode)
virtual ~AdmCallbacks ()
```

### `REngineParams`

- **Rôle** : This class encapsulates information needed to instantiate a REngine object. It contains configuration information specifying which instance of an R | Trade Execution Platform to connect to, as well as identifying information about the application instantiating an REngine . It also passes administrative callbacks to the REngine , which may invoke them to convey alerts coming from the infrastructure.
- **Include** : `#include <RApiPlus.h>`

**Public Member Functions**
```cpp
REngineParams ()
virtual ~REngineParams ()
```
**Public Attributes**
```cpp
char ** envp
AdmCallbacks * pAdmCallbacks
void * pContext
tsNCharcb sAppName
tsNCharcb sAppVersion
tsNCharcb sLogFilePath
```

### `LoginParams`

- **Rôle** : This class encapsulates information needed to login. The information includes authentication credentials, connect points for accessing services, as well as callbacks for receiving notifications from the login session. Connect point values are typically provided by the Rithmic operations team in 'connection_params.txt' files downloaded from your Rithmic download directory. See also REngine::login
- **Include** : `#include <RApiPlus.h>`

**Public Member Functions**
```cpp
LoginParams ()
virtual ~LoginParams ()
```
**Public Attributes**
```cpp
RCallbacks * pCallbacks
tsNCharcb sIhCnnctPt
tsNCharcb sIhEnvKey
tsNCharcb sIhPassword
tsNCharcb sIhUser
tsNCharcb sMdCnnctPt
tsNCharcb sMdEnvKey
tsNCharcb sMdPassword
tsNCharcb sMdUser
tsNCharcb sPnlCnnctPt
tsNCharcb sTsCnnctPt
tsNCharcb sTsEnvKey
tsNCharcb sTsPassword
tsNCharcb sTsUser
```

### `SearchTerm`

- **Rôle** : This class represents a search term for REngine::searchInstrument() . One or more instances of the SearchTerm class is passed into REngine::searchInstrument() to specify a filter when searching for instruments.
- **Include** : `#include <RApiPlus.h>`

**Public Attributes**
```cpp
bool bCaseSensitive
SearchField eField
SearchOperator eOperator
tsNCharcb sTerm
```

### `OmneException`

- **Rôle** : Exception class used by the RApi . An OmneException presents an error code and a corresponding error string.
- **Include** : `#include <RApiPlus.h>`

**Public Member Functions**
```cpp
int getErrorCode ()
const char * getErrorString ()
OmneException (int iError)
virtual ~OmneException ()
```

### `OmneTimerCB`

- **Rôle** : Timer routine class used by the RApi . An OmneTimerCB is a callback routine fired for RApi timer events. See also REngine::addTimer REngine::removeTimer REngine::startTimer REngine::stopTimer
- **Include** : `#include <RApiPlus.h>`

**Public Member Functions**
```cpp
virtual int call (tsNCharcb *pTimerName, void *pContext, int *aiCode)=0
virtual ~OmneTimerCB ()
```

## 4. Relations entre composants

- **`REngine`** : centre opérationnel. Dépendances ou points de contact : `REngineParams`, `LoginParams`, `RCallbacks`, `AdmCallbacks`, `OmneTimerCB`.
- **`LoginParams`** : porte les identifiants, connect points et callback utilisateur. Dépendances ou points de contact : `RCallbacks`, `REngine::login`.
- **`REngineParams`** : paramètre l’instanciation moteur et les callbacks administratifs. Dépendances ou points de contact : `AdmCallbacks`, `REngine`.
- **`RCallbacks`** : reçoit les événements métiers asynchrones. Dépendances ou points de contact : `*Info`, `*Report`, `*ReplayInfo`.
- **`SearchTerm`** : alimente les requêtes instrument/recherche. Dépendances ou points de contact : `SearchField`, `SearchOperator`.
- **`OrderParams`** : base commune des ordres. Dépendances ou points de contact : `LimitOrderParams`, `MarketOrderParams`, `StopLimitOrderParams`, `StopMarketOrderParams`, `ModifyOrderParams`.


## 5. Espaces de noms, fichiers et conventions

### Espaces de noms repérés

- `RApi` : espace de noms principal.
- `OmneEngineSpace` : callbacks timer annexes.
- `OmneException` : exception exposée côté API.

### Fichier source documenté

- `RApiPlus.h` : point d’entrée documentaire principal et fichier d’inclusion commun.

### Énumérations principales

#### `RApi::SearchField`
```cpp

```

#### `RApi::SearchOperator`
```cpp

```


---

# Paramètres de requête, d’initialisation et cycle de vie

## Table des matières
- [Noyau](#noyau)

## Noyau

### `AdmCallbacks`

This abstract class contains all the admin-related callbacks for RApi . A subclass of AdmCallbacks can be defined, instantiated and passed via REngineParams::pCallbacks when logging into the infrastructure. While the REngine instance is alive, the AdmCallbacks instance referenced by REngineParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AdmCallbacks.html`

**Public Member Functions**
```cpp
AdmCallbacks ()
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)=0
virtual int Environment (EnvironmentInfo *pInfo, void *pContext, int *aiCode)
virtual int EnvironmentList (EnvironmentListInfo *pInfo, void *pContext, int *aiCode)
virtual ~AdmCallbacks ()
```

### `LoginParams`

This class encapsulates information needed to login. The information includes authentication credentials, connect points for accessing services, as well as callbacks for receiving notifications from the login session. Connect point values are typically provided by the Rithmic operations team in 'connection_params.txt' files downloaded from your Rithmic download directory. See also REngine::login

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LoginParams.html`

**Public Member Functions**
```cpp
LoginParams ()
virtual ~LoginParams ()
```
**Public Attributes**
```cpp
RCallbacks * pCallbacks
tsNCharcb sIhCnnctPt
tsNCharcb sIhEnvKey
tsNCharcb sIhPassword
tsNCharcb sIhUser
tsNCharcb sMdCnnctPt
tsNCharcb sMdEnvKey
tsNCharcb sMdPassword
tsNCharcb sMdUser
tsNCharcb sPnlCnnctPt
tsNCharcb sTsCnnctPt
tsNCharcb sTsEnvKey
tsNCharcb sTsPassword
tsNCharcb sTsUser
```

### `OmneException`

Exception class used by the RApi . An OmneException presents an error code and a corresponding error string.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classOmneException.html`

**Public Member Functions**
```cpp
int getErrorCode ()
const char * getErrorString ()
OmneException (int iError)
virtual ~OmneException ()
```

### `OmneTimerCB`

Timer routine class used by the RApi . An OmneTimerCB is a callback routine fired for RApi timer events. See also REngine::addTimer REngine::removeTimer REngine::startTimer REngine::stopTimer

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classOmneEngineSpace_1_1OmneTimerCB.html`

**Public Member Functions**
```cpp
virtual int call (tsNCharcb *pTimerName, void *pContext, int *aiCode)=0
virtual ~OmneTimerCB ()
```

### `RCallbacks`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int BestBidAskQuote (BidInfo *pBid, AskInfo *pAsk, void *pContext, int *aiCode)
virtual int BestBidQuote (BidInfo *pInfo, void *pContext, int *aiCode)
virtual int BidQuote (BidInfo *pInfo, void *pContext, int *aiCode)
virtual int BinaryContractList (BinaryContractListInfo *pInfo, void *pContext, int *aiCode)
virtual int BracketReplay (BracketReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BracketTierModify (BracketTierModifyInfo *pInfo, void *pContext, int *aiCode)
virtual int BracketUpdate (BracketInfo *pInfo, void *pContext, int *aiCode)
virtual int BustReport (OrderBustReport *pReport, void *pContext, int *aiCode)
virtual int CancelReport (OrderCancelReport *pReport, void *pContext, int *aiCode)
virtual int CloseMidPrice (CloseMidPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int ClosePrice (ClosePriceInfo *pInfo, void *pContext, int *aiCode)
virtual int ClosingIndicator (ClosingIndicatorInfo *pInfo, void *pContext, int *aiCode)
virtual int Dbo (DboInfo *pInfo, void *pContext, int *aiCode)
virtual int DboBookRebuild (DboBookRebuildInfo *pInfo, void *pContext, int *aiCode)
virtual int EasyToBorrow (EasyToBorrowInfo *pInfo, void *pContext, int *aiCode)
virtual int EasyToBorrowList (EasyToBorrowListInfo *pInfo, void *pContext, int *aiCode)
virtual int EndQuote (EndQuoteInfo *pInfo, void *pContext, int *aiCode)
virtual int EquityOptionStrategyList (EquityOptionStrategyListInfo *pInfo, void *pContext, int *aiCode)
virtual int ExchangeList (ExchangeListInfo *pInfo, void *pContext, int *aiCode)
virtual int ExecutionReplay (ExecutionReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int FailureReport (OrderFailureReport *pReport, void *pContext, int *aiCode)
virtual int FillReport (OrderFillReport *pReport, void *pContext, int *aiCode)
virtual int HighBidPrice (HighBidPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int HighPrice (HighPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int HighPriceLimit (HighPriceLimitInfo *pInfo, void *pContext, int *aiCode)
virtual int IbList (IbListInfo *pInfo, void *pContext, int *aiCode)
virtual int InstrumentByUnderlying (InstrumentByUnderlyingInfo *pInfo, void *pContext, int *aiCode)
virtual int InstrumentSearch (InstrumentSearchInfo *pInfo, void *pContext, int *aiCode)
virtual int LimitOrderBook (LimitOrderBookInfo *pInfo, void *pContext, int *aiCode)
virtual int LineUpdate (LineInfo *pInfo, void *pContext, int *aiCode)
virtual int LowAskPrice (LowAskPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int LowPrice (LowPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int LowPriceLimit (LowPriceLimitInfo *pInfo, void *pContext, int *aiCode)
virtual int MarketMode (MarketModeInfo *pInfo, void *pContext, int *aiCode)
virtual int MidPrice (MidPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int ModifyReport (OrderModifyReport *pReport, void *pContext, int *aiCode)
virtual int NotCancelledReport (OrderNotCancelledReport *pReport, void *pContext, int *aiCode)
virtual int NotModifiedReport (OrderNotModifiedReport *pReport, void *pContext, int *aiCode)
virtual int OpeningIndicator (OpeningIndicatorInfo *pInfo, void *pContext, int *aiCode)
virtual int OpenInterest (OpenInterestInfo *pInfo, void *pContext, int *aiCode)
virtual int OpenOrderReplay (OrderReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int OpenPrice (OpenPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int OptionList (OptionListInfo *pInfo, void *pContext, int *aiCode)
virtual int OrderHistoryDates (OrderHistoryDatesInfo *pInfo, void *pContext, int *aiCode)
virtual int OrderReplay (OrderReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int OtherReport (OrderReport *pReport, void *pContext, int *aiCode)
virtual int PasswordChange (PasswordChangeInfo *pInfo, void *pContext, int *aiCode)
virtual int PnlReplay (PnlReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int PnlUpdate (PnlInfo *pInfo, void *pContext, int *aiCode)
virtual int PositionExit (PositionExitInfo *pInfo, void *pContext, int *aiCode)
virtual int PriceIncrUpdate (PriceIncrInfo *pInfo, void *pContext, int *aiCode)
virtual int ProductRmsList (ProductRmsListInfo *pInfo, void *pContext, int *aiCode)
virtual int ProjectedSettlementPrice (ProjectedSettlementPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int Quote (QuoteInfo *pInfo, void *pContext, int *aiCode)
virtual int Quote (QuoteReport *pReport, void *pContext, int *aiCode)
virtual int QuoteReplay (QuoteReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int RefData (RefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int RejectReport (OrderRejectReport *pReport, void *pContext, int *aiCode)
virtual int SettlementPrice (SettlementPriceInfo *pInfo, void *pContext, int *aiCode)
virtual int SingleOrderReplay (SingleOrderReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int SodUpdate (SodReport *pReport, void *pContext, int *aiCode)
virtual int StatusReport (OrderStatusReport *pReport, void *pContext, int *aiCode)
virtual int Strategy (StrategyInfo *pInfo, void *pContext, int *aiCode)
virtual int StrategyList (StrategyListInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeCondition (TradeInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeCorrectReport (OrderTradeCorrectReport *pReport, void *pContext, int *aiCode)
virtual int TradePrint (TradeInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeReplay (TradeReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeRoute (TradeRouteInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeRouteList (TradeRouteListInfo *pInfo, void *pContext, int *aiCode)
virtual int TradeVolume (TradeVolumeInfo *pInfo, void *pContext, int *aiCode)
virtual int TriggerPulledReport (OrderTriggerPulledReport *pReport, void *pContext, int *aiCode)
virtual int TriggerReport (OrderTriggerReport *pReport, void *pContext, int *aiCode)
virtual int User (UserInfo *pInfo, void *pContext, int *aiCode)
virtual int UserDefinedSpreadCreate (UserDefinedSpreadCreateInfo *pInfo, void *pContext, int *aiCode)
virtual int UserList (UserListInfo *pInfo, void *pContext, int *aiCode)
virtual int UserProfile (UserProfileInfo *pInfo, void *pContext, int *aiCode)
virtual int VolumeAtPrice (VolumeAtPriceInfo *pInfo, void *pContext, int *aiCode)
virtual ~RCallbacks ()
```

### `REngine`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
int getAppVersion (tsNCharcb *pAppVersion, int *aiCode)
int getAuxRefData (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int getCallbacks (RCallbacks **apCallbacks, int *aiCode)
int getContext (void **apContext, int *aiCode)
int getEasyToBorrowList (void *pContext, int *aiCode)
int getEnvironment (tsNCharcb *pKey, void *pContext, int *aiCode)
int getEquityOptionStrategyList (tsNCharcb *pExchange, tsNCharcb *pUnderlying, tsNCharcb *pStrategyType, tsNCharcb *pExpiration, void *pContext, int *aiCode)
int getIhCnnctPt (tsNCharcb *pIhCnnctPt, int *aiCode)
int getInstrumentByUnderlying (tsNCharcb *pUnderlying, tsNCharcb *pExchange, tsNCharcb *pExpiration, void *pContext, int *aiCode)
int getMdCnnctPt (tsNCharcb *pMdCnnctPt, int *aiCode)
int getOmne (void **apOmne, int *aiCode)
int getOptionList (tsNCharcb *pExchange, tsNCharcb *pProductCode, tsNCharcb *pExpirationCCYYMM, void *pContext, int *aiCode)
int getOrderContext (tsNCharcb *pOrderNum, void **apContext, int *aiCode)
int getPassword (tsNCharcb *pPassword, int *aiCode)
int getPendingInputSize (int iConnId, unsigned long int *aulPendingBytes, int *aiCode)
int getPnlCnnctPt (tsNCharcb *pPnlCnnctPt, int *aiCode)
int getPriceIncrInfo (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int getProductRmsInfo (AccountInfo *pAccount, int *aiCode)
int getRefData (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int getStrategyInfo (tsNCharcb *pExchange, tsNCharcb *pTicker, void *pContext, int *aiCode)
int getStrategyList (tsNCharcb *pExchange, tsNCharcb *pProductCode, tsNCharcb *pStrategyType, tsNCharcb *pExpirationCCYYMM, void *pContext, int *aiCode)
int getTsCnnctPt (tsNCharcb *pTsCnnctPt, int *aiCode)
int getUser (tsNCharcb *pUser, int *aiCode)
int getUserProfile (int iConnId, void *pContext, int *aiCode)
int getVolumeAtPrice (tsNCharcb *pExchange, tsNCharcb *pTicker, void *pContext, int *aiCode)
int isThereAnAggregator (int *aiCode)
int linkOrders (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, int iArrayLen, int *aiCode)
int listAgreements (bool bAccepted, void *pContext, int *aiCode)
int listAssignedUsers (AccountInfo *pAccount, void *pContext, int *aiCode)
int listBinaryContracts (tsNCharcb *pExchange, tsNCharcb *pProductCode, void *pContext, int *aiCode)
int listEnvironments (void *pContext, int *aiCode)
int listExchanges (void *pContext, int *aiCode)
int listIbs (void *pContext, int *aiCode)
int listOrderHistoryDates (void *pContext, int *aiCode)
int listTradeRoutes (void *pContext, int *aiCode)
int listUsers (tsNCharcb *pIbId, tsNCharcb *pUserType, void *pContext, int *aiCode)
int lock (int *aiCode)
int login (LoginParams *pParams, int *aiCode)
int loginRepository (tsNCharcb *pEnvKey, tsNCharcb *pUser, tsNCharcb *pPassword, tsNCharcb *pCnnctPt, RCallbacks *pCallbacks, int *aiCode)
int logout (int *aiCode)
int logoutRepository (int *aiCode)
int modifyBracketTier (AccountInfo *pAccount, tsNCharcb *pOrderNum, bool bTarget, int iOldTickOffset, int iNewTickOffset, void *pContext, int *aiCode)
int modifyOrder (ModifyLimitOrderParams *pParams, int *aiCode)
int modifyOrder (ModifyOrderParams *pParams, int *aiCode)
int modifyOrder (ModifyStopLimitOrderParams *pParams, int *aiCode)
int modifyOrder (ModifyStopMarketOrderParams *pParams, int *aiCode)
int modifyOrderList (ModifyOrderParams *pParams, int iArrayLen, int *aiCode)
int modifyOrderRefData (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pUserMsg, tsNCharcb *pUserTag, int *aiCode)
int rebuildBook (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int rebuildDboBook (tsNCharcb *pExchange, tsNCharcb *pTicker, bool bPrice, double dPrice, void *pContext, int *aiCode)
int removeTimer (tsNCharcb *const pTimerName, int *aiCode)
REngine (REngineParams *pParams)
int replayAllOrders (AccountInfo *pAccount, int iStartSsboe, int iEndSsboe, int *aiCode)
int replayBars (ReplayBarParams *pParams, int *aiCode)
int replayBrackets (AccountInfo *pAccount, int *aiCode)
int replayExecutions (AccountInfo *pAccount, int iStartSsboe, int iEndSsboe, int *aiCode)
int replayHistoricalOrders (AccountInfo *pAccount, tsNCharcb *pDate, int *aiCode)
int replayOpenOrders (AccountInfo *pAccount, int *aiCode)
int replayPnl (AccountInfo *pAccount, int *aiCode)
int replayQuotes (AccountInfo *pAccount, void *pContext, int *aiCode)
int replaySingleHistoricalOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pDate, void *pContext, int *aiCode)
int replaySingleOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, void *pContext, int *aiCode)
int replayTrades (tsNCharcb *pExchange, tsNCharcb *pTicker, int iStartSsboe, int iEndSsboe, int *aiCode)
int resumeInput (int *aiCode)
int searchInstrument (tsNCharcb *pExchange, SearchTerm *asTerms, int iNumTerms, void *pContext, int *aiCode)
int sendBracketOrder (OrderParams *pEntry, BracketParams *pBracketParams, int *aiCode)
int sendBracketOrder (OrderParams *pEntry, tsNCharcb *pBracketType, tsNCharcb *pOrderOperationType, bool bTrailingStop, int iTrailByPriceId, int iTargetArrayLen, int *pTargetQty, int *pTargetTicks, int iStopArrayLen, int *pStopQty, int *pStopTicks, int *aiCode)
int sendOcaList (tsNCharcb *pOcaType, OrderParams *pParamsArray, int iArrayLen, int *aiCode)
int sendOrder (LimitOrderParams *pParams, int *aiCode)
int sendOrder (MarketOrderParams *pParams, int *aiCode)
int sendOrder (StopLimitOrderParams *pParams, int *aiCode)
int sendOrder (StopMarketOrderParams *pParams, int *aiCode)
int sendOrderList (OrderParams *pParamsArray, int iArrayLen, int *aiCode)
int setContext (void *pContext, int *aiCode)
int setEnvironmentVariable (tsNCharcb *pKey, tsNCharcb *pVariable, tsNCharcb *pValue, int *aiCode)
int setOrderContext (tsNCharcb *pOrderNum, void *pContext, int *aiCode)
int startTimer (tsNCharcb *const pTimerName, int *aiCode)
int stopTimer (tsNCharcb *const pTimerName, int *aiCode)
int submitQuoteList (QuoteParams *pParams, int iArrayLen, int *aiCode)
int subscribe (tsNCharcb *pExchange, tsNCharcb *pTicker, int iFlags, int *aiCode)
int subscribeAutoLiquidate (AccountInfo *pAccount, int *aiCode)
int subscribeBar (BarParams *pParams, int *aiCode)
int subscribeBracket (AccountInfo *pAccount, int *aiCode)
int subscribeByUnderlying (tsNCharcb *pUnderlying, tsNCharcb *pExchange, tsNCharcb *pExpiration, int iFlags, void *pContext, int *aiCode)
int subscribeDbo (tsNCharcb *pExchange, tsNCharcb *pTicker, bool bPrice, double dPrice, void *pContext, int *aiCode)
int subscribeEasyToBorrow (void *pContext, int *aiCode)
int subscribeOrder (AccountInfo *pAccount, int *aiCode)
int subscribePnl (AccountInfo *pAccount, int *aiCode)
int subscribeTradeRoute (tsNCharcb *pFcmId, tsNCharcb *pIbId, void *pContext, int *aiCode)
int subscribeUser (tsNCharcb *pIbId, void *pContext, int *aiCode)
int suspendInput (int *aiCode)
int tryLock (int *aiCode)
int unlock (int *aiCode)
int unsetEnvironmentVariable (tsNCharcb *pKey, tsNCharcb *pVariable, int *aiCode)
int unsubscribe (tsNCharcb *pExchange, tsNCharcb *pTicker, int *aiCode)
int unsubscribeAutoLiquidate (AccountInfo *pAccount, int *aiCode)
int unsubscribeBar (BarParams *pParams, int *aiCode)
int unsubscribeBracket (AccountInfo *pAccount, int *aiCode)
int unsubscribeByUnderlying (tsNCharcb *pUnderlying, tsNCharcb *pExchange, tsNCharcb *pExpiration, int *aiCode)
int unsubscribeDbo (tsNCharcb *pExchange, tsNCharcb *pTicker, bool bPrice, double dPrice, void *pContext, int *aiCode)
int unsubscribeEasyToBorrow (int *aiCode)
int unsubscribeOrder (AccountInfo *pAccount, int *aiCode)
int unsubscribePnl (AccountInfo *pAccount, int *aiCode)
int unsubscribeTradeRoute (tsNCharcb *pFcmId, tsNCharcb *pIbId, int *aiCode)
int unsubscribeUser (tsNCharcb *pIbId, int *aiCode)
virtual ~REngine ()
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngineParams`

This class encapsulates information needed to instantiate a REngine object. It contains configuration information specifying which instance of an R | Trade Execution Platform to connect to, as well as identifying information about the application instantiating an REngine . It also passes administrative callbacks to the REngine , which may invoke them to convey alerts coming from the infrastructure.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngineParams.html`

**Public Member Functions**
```cpp
REngineParams ()
virtual ~REngineParams ()
```
**Public Attributes**
```cpp
char ** envp
AdmCallbacks * pAdmCallbacks
void * pContext
tsNCharcb sAppName
tsNCharcb sAppVersion
tsNCharcb sLogFilePath
```

### `SearchTerm`

This class represents a search term for REngine::searchInstrument() . One or more instances of the SearchTerm class is passed into REngine::searchInstrument() to specify a filter when searching for instruments.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1SearchTerm.html`

**Public Attributes**
```cpp
bool bCaseSensitive
SearchField eField
SearchOperator eOperator
tsNCharcb sTerm
```


---

# Données de marché, référentiel et historique

## Table des matières
- [Données de marché et référentiel](#donnees-de-marche-et-referentiel)

## Données de marché et référentiel

### `AskInfo`

This class represents an ask quote. The AskInfo class is used to pass quote information received based on REngine::subscribe() requests via the RCallbacks::AskQuote() and RCallbacks::BestAskQuote callbacks.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AskInfo.html`

**Public Member Functions**
```cpp
AskInfo ()
int dump (int *aiCode)
virtual ~AskInfo ()
```
**Public Attributes**
```cpp
bool bLeanPriceFlag
bool bPriceFlag
bool bSizeFlag
double dLeanPrice
double dPrice
int iConnId
int iNumOrders
int iSsboe
int iType
int iUpdateType
int iUsecs
long long llImpliedSize
// ... 3 éléments supplémentaires
```

### `AuxRefDataInfo`

This class represents auxiliary reference data for an instrument. The AuxRefDataInfo class is used to pass auxiliary reference data resulting from calls to REngine::getAuxRefData() or REngine::subscribe() where the flags include RApi::MD_REF_DATA . The data is passed via the callback RCallbacks::AuxRefData() . Please note that this auxiliary reference data is seperate and distinct from RefDataInfo and PriceIncrInfo .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AuxRefDataInfo.html`

**Public Member Functions**
```cpp
AuxRefDataInfo ()
int dump (int *aiCode)
virtual ~AuxRefDataInfo ()
```
**Public Attributes**
```cpp
bool bUnitOfMeasureQty
double dUnitOfMeasureQty
int iRpCode
int iSsboe
int iType
tsNCharcb sExchange
tsNCharcb sFirstDeliveryDate
tsNCharcb sFirstNoticeDate
tsNCharcb sFirstPositionDate
tsNCharcb sFirstTradingDate
tsNCharcb sLastDeliveryDate
tsNCharcb sLastNoticeDate
// ... 5 éléments supplémentaires
```

### `BarInfo`

An instance of this class represents bar/candlestick data. The BarInfo class holds bar/candlestick data, consisting mainly of open/high/low/close trade prices over a given set of market data trades. There are different types of bars (different ways of aggregating the set of trades), including by time period, price range, volume traded, and number of trades. One can subscribe to bars from the real-time stream using REngine::subscribeBar() , or replay bars from a historical database, using REngine::replayBars() . See also REngine::replayBars() REngine::subscribeBar()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BarInfo.html`

**Public Member Functions**
```cpp
BarInfo ()
int dump (int *aiCode)
virtual ~BarInfo ()
```
**Public Attributes**
```cpp
double * adProfilePrices
int * aiProfileBuyerAsAggressorNumTrades
int * aiProfileNoAggressorNumTrades
int * aiProfileSellerAsAggressorNumTrades
long long * allProfileBuyerAsAggressorVolumes
long long * allProfileNoAggressorVolumes
long long * allProfileSellerAsAggressorVolumes
bool bBuyerAsAggressorVolume
bool bCustomSession
bool bSellerAsAggressorVolume
bool bSettlementPrice
bool bVwap
// ... 29 éléments supplémentaires
```

### `BarInfo::dSpecifiedRange`

An instance of this class represents bar/candlestick data. The BarInfo class holds bar/candlestick data, consisting mainly of open/high/low/close trade prices over a given set of market data trades. There are different types of bars (different ways of aggregating the set of trades), including by time period, price range, volume traded, and number of trades. One can subscribe to bars from the real-time stream using REngine::subscribeBar() , or replay bars from a historical database, using REngine::replayBars() . See also REngine::replayBars() REngine::subscribeBar()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BarInfo.html`

**Public Member Functions**
```cpp
BarInfo ()
int dump (int *aiCode)
virtual ~BarInfo ()
```
**Public Attributes**
```cpp
double * adProfilePrices
int * aiProfileBuyerAsAggressorNumTrades
int * aiProfileNoAggressorNumTrades
int * aiProfileSellerAsAggressorNumTrades
long long * allProfileBuyerAsAggressorVolumes
long long * allProfileNoAggressorVolumes
long long * allProfileSellerAsAggressorVolumes
bool bBuyerAsAggressorVolume
bool bCustomSession
bool bSellerAsAggressorVolume
bool bSettlementPrice
bool bVwap
// ... 29 éléments supplémentaires
```

### `BarInfo::iSpecifiedMinutes`

An instance of this class represents bar/candlestick data. The BarInfo class holds bar/candlestick data, consisting mainly of open/high/low/close trade prices over a given set of market data trades. There are different types of bars (different ways of aggregating the set of trades), including by time period, price range, volume traded, and number of trades. One can subscribe to bars from the real-time stream using REngine::subscribeBar() , or replay bars from a historical database, using REngine::replayBars() . See also REngine::replayBars() REngine::subscribeBar()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BarInfo.html`

**Public Member Functions**
```cpp
BarInfo ()
int dump (int *aiCode)
virtual ~BarInfo ()
```
**Public Attributes**
```cpp
double * adProfilePrices
int * aiProfileBuyerAsAggressorNumTrades
int * aiProfileNoAggressorNumTrades
int * aiProfileSellerAsAggressorNumTrades
long long * allProfileBuyerAsAggressorVolumes
long long * allProfileNoAggressorVolumes
long long * allProfileSellerAsAggressorVolumes
bool bBuyerAsAggressorVolume
bool bCustomSession
bool bSellerAsAggressorVolume
bool bSettlementPrice
bool bVwap
// ... 29 éléments supplémentaires
```

### `BarInfo::iSpecifiedTicks`

An instance of this class represents bar/candlestick data. The BarInfo class holds bar/candlestick data, consisting mainly of open/high/low/close trade prices over a given set of market data trades. There are different types of bars (different ways of aggregating the set of trades), including by time period, price range, volume traded, and number of trades. One can subscribe to bars from the real-time stream using REngine::subscribeBar() , or replay bars from a historical database, using REngine::replayBars() . See also REngine::replayBars() REngine::subscribeBar()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BarInfo.html`

**Public Member Functions**
```cpp
BarInfo ()
int dump (int *aiCode)
virtual ~BarInfo ()
```
**Public Attributes**
```cpp
double * adProfilePrices
int * aiProfileBuyerAsAggressorNumTrades
int * aiProfileNoAggressorNumTrades
int * aiProfileSellerAsAggressorNumTrades
long long * allProfileBuyerAsAggressorVolumes
long long * allProfileNoAggressorVolumes
long long * allProfileSellerAsAggressorVolumes
bool bBuyerAsAggressorVolume
bool bCustomSession
bool bSellerAsAggressorVolume
bool bSettlementPrice
bool bVwap
// ... 29 éléments supplémentaires
```

### `BarInfo::sSpecifiedDate`

An instance of this class represents bar/candlestick data. The BarInfo class holds bar/candlestick data, consisting mainly of open/high/low/close trade prices over a given set of market data trades. There are different types of bars (different ways of aggregating the set of trades), including by time period, price range, volume traded, and number of trades. One can subscribe to bars from the real-time stream using REngine::subscribeBar() , or replay bars from a historical database, using REngine::replayBars() . See also REngine::replayBars() REngine::subscribeBar()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BarInfo.html`

**Public Member Functions**
```cpp
BarInfo ()
int dump (int *aiCode)
virtual ~BarInfo ()
```
**Public Attributes**
```cpp
double * adProfilePrices
int * aiProfileBuyerAsAggressorNumTrades
int * aiProfileNoAggressorNumTrades
int * aiProfileSellerAsAggressorNumTrades
long long * allProfileBuyerAsAggressorVolumes
long long * allProfileNoAggressorVolumes
long long * allProfileSellerAsAggressorVolumes
bool bBuyerAsAggressorVolume
bool bCustomSession
bool bSellerAsAggressorVolume
bool bSettlementPrice
bool bVwap
// ... 29 éléments supplémentaires
```

### `BarParams`

This class encapsulates information needed to request data bars.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BarParams.html`

**Public Member Functions**
```cpp
BarParams ()
virtual ~BarParams ()
```
**Public Attributes**
```cpp
bool bCustomSession
bool bProfile
double dSpecifiedRange
int iCloseSsm
int iOpenSsm
int iSpecifiedMinutes
int iSpecifiedSeconds
int iSpecifiedTicks
int iType
long long llSpecifiedVolume
void * pContext
tsNCharcb sExchange
// ... 1 éléments supplémentaires
```

### `BarReplayInfo`

This class demarcates the end of a stream of historical bar data. The BarReplayInfo class is used to convey the conclusion of the data stream and the response code associated with a call to REngine::replayBars() . When the data stream concludes, RCallbacks::BarReplay() is invoked, passing a BarReplayInfo instance containing the parameter values initially passed into the corresponding call to REngine::replayBars() . If the returned bars does not include data for the entire requested time period, and/or if the number of returned bars is a round number (such as 10000), then it is possible that the request was truncated. One can request the missing bars by shifting the time period of the original request to cover the truncated data. This truncation can occur when large amounts of data are requested. See also REngine::replayBars() RCallbacks::Bar() RCallbacks::BarReplay()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BarReplayInfo.html`

**Public Member Functions**
```cpp
BarReplayInfo ()
int dump (int *aiCode)
virtual ~BarReplayInfo ()
```
**Public Attributes**
```cpp
bool bCustomSession
bool bProfile
double dSpecifiedRange
int iCloseSsm
int iEndSsboe
int iEndUsecs
int iOpenSsm
int iRpCode
int iSpecifiedMinutes
int iSpecifiedSeconds
int iSpecifiedTicks
int iStartSsboe
// ... 9 éléments supplémentaires
```

### `BidInfo`

This class represents an bid quote. The BidInfo class is used to pass quote information received based on REngine::subscribe() requests via the RCallbacks::BidQuote() and RCallbacks::BestBidQuote callbacks.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BidInfo.html`

**Public Member Functions**
```cpp
BidInfo ()
int dump (int *aiCode)
virtual ~BidInfo ()
```
**Public Attributes**
```cpp
bool bLeanPriceFlag
bool bPriceFlag
bool bSizeFlag
double dLeanPrice
double dPrice
int iConnId
int iNumOrders
int iSsboe
int iType
int iUpdateType
int iUsecs
long long llImpliedSize
// ... 3 éléments supplémentaires
```

### `CloseMidPriceInfo`

This class represents close mid price information. The CloseMidPriceInfo class is used to convey the close mid price for a trading session. To express interest in mid price data, one can specify the MD_MID_PRICE subscription flag when subscribing. The MidPriceInfo class is used to pass mid price info via RCallbacks::MidPrice() and the CloseMidPriceInfo class is used to convey close mid prices via RCallbacks::CloseMidPrice() . See also REngine::subscribe RApi::MD_MID_PRICE MidPriceInfo RCallbacks::MidPrice CloseMidPriceInfo RCallbacks::CloseMidPrice

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1CloseMidPriceInfo.html`

**Public Member Functions**
```cpp
CloseMidPriceInfo ()
int dump (int *aiCode)
virtual ~CloseMidPriceInfo ()
```
**Public Attributes**
```cpp
bool bClosePrice
double dClosePrice
int iSsboe
int iType
int iUsecs
tsNCharcb sClosePriceDate
tsNCharcb sExchange
tsNCharcb sTicker
```

### `ClosePriceInfo`

This class represents close information. The ClosePriceInfo class is used to pass close price information received as a result of calls to REngine::subscribe() via RCallbacks::ClosePrice() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ClosePriceInfo.html`

**Public Member Functions**
```cpp
ClosePriceInfo ()
int dump (int *aiCode)
virtual ~ClosePriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `DboBookRebuildInfo`

This class represents status information related to requesting an dbo order book image. DboBookRebuildInfo conveys the conclusion and/or response code associated with a call to REngine::rebuildDboBook() . The individual orders comprising the order book image will be conveyed via RCallbacks::Dbo() . See also REngine::rebuildDboBook() RCallbacks::Dbo() RCallbacks::DboBookRebuild()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1DboBookRebuildInfo.html`

**Public Member Functions**
```cpp
DboBookRebuildInfo ()
int dump (int *aiCode)
virtual ~DboBookRebuildInfo ()
```
**Public Attributes**
```cpp
bool bPrice
double dPrice
int iRpCode
void * pContext
tsNCharcb sExchange
tsNCharcb sRpCode
tsNCharcb sTicker
```

### `DboInfo`

This class represents depth-by-order (DBO) market data. What is DBO data and how might I use it? See also REngine::subscribeDbo REngine::rebuildDboBook RCallbacks::Dbo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1DboInfo.html`

**Public Member Functions**
```cpp
DboInfo ()
int dump (int *aiCode)
virtual ~DboInfo ()
```
**Public Attributes**
```cpp
bool bPreviousPrice
bool bPrice
bool bSize
double dPreviousPrice
double dPrice
int iJopNsecs
int iJopSsboe
int iSourceNsecs
int iSourceSsboe
int iSourceUsecs
int iSsboe
int iType
// ... 9 éléments supplémentaires
```

### `EndQuoteInfo`

This class represents the end to a set of updates to an order book. The EndQuoteInfo class conveys that a set of quote updates to the instrument's order book is over, and the order book can be evaluated. This class is similar to the AskInfo or BidInfo classes where iUpdateType is UPDATE_TYPE_END, except that this class does not include price level information.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1EndQuoteInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
EndQuoteInfo ()
virtual ~EndQuoteInfo ()
```
**Public Attributes**
```cpp
int iConnId
int iSsboe
int iType
int iUpdateType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `EquityOptionStrategyListInfo`

An instance of this class is the response to a call to REngine::getEquityOptionStrategyList() . An EquityOptionStrategyListInfo object is used to pass : a list of exchanges, a list of underlying instruments within an exchange, a list of strategy types within an underlying (within an exchange), a list of expiration dates within a strategy type (within an underlying/exchange), or a list of tickers representing strategies within an expiration date What is returned depends upon what was initially passed into REngine::getEquityOptionStrategyList() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1EquityOptionStrategyListInfo.html`

**Public Member Functions**
```cpp
virtual int clearHandles (int *aiCode)
int dump (int *aiCode)
EquityOptionStrategyListInfo ()
virtual ~EquityOptionStrategyListInfo ()
```
**Public Attributes**
```cpp
int iExchangeArrayLen
int iExpirationArrayLen
int iRpCode
int iStrategyTypeArrayLen
int iTickerArrayLen
int iUnderlyingArrayLen
void * pContext
tsNCharcb * pExchangeArray
tsNCharcb * pExpirationArray
tsNCharcb * pStrategyTypeArray
tsNCharcb * pTickerArray
tsNCharcb * pUnderlyingArray
```

### `ExchangeListInfo`

An instance of this class represents a list of exchanges. An ExchangeListInfo object is used to pass an array of tsNCharcbs via RCallbacks::ExchangeList() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ExchangeListInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
ExchangeListInfo ()
virtual ~ExchangeListInfo ()
```
**Public Attributes**
```cpp
tsNCharcb * asExchangeArray
int iArrayLen
int iRpCode
void * pContext
tsNCharcb sRpCode
```

### `HighBidPriceInfo`

This class represents high bid price information. The HighBidPriceInfo class is used to pass the current trading session's highest bid price. One can subscribe to highest bid price data by calling REngine::subscribe() with the MD_HIGH_BID_LOW_ASK flag enabled. Instances of this class will be passed via RCallbacks::HighBidPrice() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1HighBidPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
HighBidPriceInfo ()
virtual ~HighBidPriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `HighPriceInfo`

This class represents high information. The HighPriceInfo class is used to pass the daily high price received as a result of calls to REngine::subscribe() requests via the RCallbacks::HighPrice() callback.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1HighPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
HighPriceInfo ()
virtual ~HighPriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `HighPriceLimitInfo`

This class represents high price limit information. The HighPriceLimitInfo class is used to convey the upper bound of allowable prices during a trading session. See also RApi::MD_PRICE_LIMIT

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1HighPriceLimitInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
HighPriceLimitInfo ()
virtual ~HighPriceLimitInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `InstrumentByUnderlyingInfo`

An instance of this class is the response to a call to REngine::getInstrumentByUnderlying() . An InstrumentByUnderlyingInfo object is used to pass : a list of underlyings, a list of expiration dates within an underlying, or a list of RefDataInfo objects representing options with an expiration date within an underlying What is returned depends upon what was initially passed into REngine::getInstrumentByUnderlying() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1InstrumentByUnderlyingInfo.html`

**Public Member Functions**
```cpp
virtual int clearHandles (int *aiCode)
int dump (int *aiCode)
InstrumentByUnderlyingInfo ()
virtual ~InstrumentByUnderlyingInfo ()
```
**Public Attributes**
```cpp
RefDataInfo * asRefDataInfoArray
int iArrayLen
int iExchangeArrayLen
int iExpirationArrayLen
int iRpCode
int iUnderlyingArrayLen
void * pContext
tsNCharcb * pExchangeArray
tsNCharcb * pExpirationArray
tsNCharcb * pUnderlyingArray
tsNCharcb sRpCode
```

### `InstrumentSearchInfo`

An instance of this class marks the conclusion of the results returned from a call to REngine::searchInstrument() . REngine::searchInstrument() returns zero to many instruments, with each returned instrument being conveyed via RCallbacks::RefData() . When the results are complete, RCallbacks::InstrumentSearch() will be invoked, with this class conveying the initial parameters. See also REngine::searchInstrument() RCallbacks::RefData() RCallbacks::InstrumentSearch()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1InstrumentSearchInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
InstrumentSearchInfo ()
virtual ~InstrumentSearchInfo ()
```
**Public Attributes**
```cpp
SearchTerm * aoTerms
RefDataInfo * asArray
int iArrayLen
int iNumTerms
int iRpCode
void * pContext
tsNCharcb sExchange
```

### `LimitOrderBookInfo`

This class represents a limit order book. This class is used to pass order books received as a result of calls to REngine::rebuildBook() or REngine::subscribe() . The callback for both of these routines is RCallbacks::LimitOrderBook() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LimitOrderBookInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
LimitOrderBookInfo ()
virtual ~LimitOrderBookInfo ()
```
**Public Attributes**
```cpp
double * adAskPriceArray
double * adBidPriceArray
int * aiAskNumOrdersArray
int * aiAskSsboeArray
int * aiAskUsecsArray
int * aiBidNumOrdersArray
int * aiBidSsboeArray
int * aiBidUsecsArray
long long * allAskImpliedSizeArray
long long * allAskSizeArray
long long * allBidImpliedSizeArray
long long * allBidSizeArray
// ... 10 éléments supplémentaires
```

### `LowAskPriceInfo`

This class represents low ask price information. The LowAskPriceInfo class is used to pass the current trading session's lowest asking price. One can subscribe to lowest asking price data by calling REngine::subscribe() with the MD_HIGH_BID_LOW_ASK flag enabled. Instances of this class will be passed via the RCallbacks::LowAskPrice() callback.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LowAskPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
LowAskPriceInfo ()
virtual ~LowAskPriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `LowPriceInfo`

This class represents low information. The LowPriceInfo class is used to pass the daily low price received as a result of calls to REngine::subscribe() requests via the RCallbacks::LowPrice() callback.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LowPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
LowPriceInfo ()
virtual ~LowPriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `LowPriceLimitInfo`

This class represents low price limit information. The LowPriceLimitInfo class is used to convey the lower bound of allowable prices during a trading session. See also RApi::MD_PRICE_LIMIT

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LowPriceLimitInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
LowPriceLimitInfo ()
virtual ~LowPriceLimitInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `MarketModeInfo`

This class represents the trading status of an instrument. The MarketModeInfo class is used to pass the trading status (market mode) of an instrument.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1MarketModeInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
MarketModeInfo ()
virtual ~MarketModeInfo ()
```
**Public Attributes**
```cpp
int iJopNsecs
int iJopSsboe
int iSourceNsecs
int iSourceSsboe
int iSourceUsecs
int iSsboe
int iType
int iUsecs
tsNCharcb sEvent
tsNCharcb sExchange
tsNCharcb sMarketMode
tsNCharcb sReason
// ... 1 éléments supplémentaires
```

### `MidPriceInfo`

This class represents mid price information. If trade market data is unavailable, mid price information may be published in its stead. The mid price is the average of the best bid and best ask, i.e. (best bid price + best ask)/2. Some additional statistics on the mid price are calculated, such as the open, high, low and close mid prices. To express interest in mid price data, one can specify the MD_MID_PRICE subscription flag when subscribing. The MidPriceInfo class is used to pass mid price info via RCallbacks::MidPrice() and the CloseMidPriceInfo class is used to convey close mid prices via RCallbacks::CloseMidPrice() . See also REngine::subscribe RApi::MD_MID_PRICE MidPriceInfo RCallbacks::MidPrice CloseMidPriceInfo RCallbacks::CloseMidPrice

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1MidPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
MidPriceInfo ()
virtual ~MidPriceInfo ()
```
**Public Attributes**
```cpp
bool bHighPrice
bool bLastPrice
bool bLowPrice
bool bNetChange
bool bOpenPrice
bool bPercentChange
double dHighPrice
double dLastPrice
double dLowPrice
double dNetChange
double dOpenPrice
double dPercentChange
// ... 5 éléments supplémentaires
```

### `OpenInterestInfo`

This class represents the open interest for an instrument.. The OpenInterestInfo class is used to pass open interest information received based on REngine::subscribe() requests via the RCallbacks::OpenInterest() callback.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OpenInterestInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OpenInterestInfo ()
virtual ~OpenInterestInfo ()
```
**Public Attributes**
```cpp
bool bQuantityFlag
int iSsboe
int iType
int iUsecs
long long llQuantity
tsNCharcb sExchange
tsNCharcb sTicker
```

### `OpenPriceInfo`

This class represents open price information as published by an exchange. The OpenPriceInfo class is used to present open price information. The method REngine::subscribe() with MD_OPEN in the subscription flags is used to express interest in receiving open prices. The callback used to pass that information is RCallbacks::OpenPrice() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OpenPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OpenPriceInfo ()
virtual ~OpenPriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `OpeningIndicatorInfo`

This class represents opening indicators. The OpeningIndicatorInfo class is used to pass opening indicator price information received as a result of calls to REngine::subscribe() requests via the RCallbacks::OpeningIndicator() callback. Not all exchanges or instruments make use of opening indicators.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OpeningIndicatorInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OpeningIndicatorInfo ()
virtual ~OpeningIndicatorInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
bool bVolumeFlag
double dPrice
int iSsboe
int iType
int iUsecs
long long llVolume
tsNCharcb sExchange
tsNCharcb sTicker
```

### `OptionListInfo`

An instance of this class is the response to a call to REngine::getOptionList() . An OptionListInfo object is used to pass : a list of exchanges, a list of product codes within an exchange, a list of expiration dates within a product code (within an exchange), or a list of RefDataInfo objects representing options within an expiration date What is returned depends upon what was initially passed into REngine::getOptionList() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OptionListInfo.html`

**Public Member Functions**
```cpp
virtual int clearHandles (int *aiCode)
int dump (int *aiCode)
OptionListInfo ()
virtual ~OptionListInfo ()
```
**Public Attributes**
```cpp
RefDataInfo * asRefDataInfoArray
int iArrayLen
int iExchangeArrayLen
int iExpirationCCYYMMArrayLen
int iProductCodeArrayLen
int iRpCode
void * pContext
tsNCharcb * pExchangeArray
tsNCharcb * pExpirationCCYYMMArray
tsNCharcb * pProductCodeArray
tsNCharcb sRpCode
```

### `OrderTradeCorrectReport`

The OrderTradeCorrectReport represents a trade which is under investigation and is passed in RCallbacks::TradeCorrectReport() . The OrderTradeCorrectReport class conveys information about a possible trade correction. See also REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderTradeCorrectReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderTradeCorrectReport ()
virtual ~OrderTradeCorrectReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
tsNCharcb sFillType
tsNCharcb sSettlementCurrency
tsNCharcb sSettlementDate
tsNCharcb sTradeDate
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
// ... 41 éléments supplémentaires
```

### `PnlInfo::dAvgOpenFillPrice`

An instance of this class represents the pnl of an Account at the Account level or for an instrument.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PnlInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
PnlInfo ()
virtual ~PnlInfo ()
```
**Public Attributes**
```cpp
bool bPossOutOfOrder
double dAccountBalance
double dAvailableBuyingPower
double dAvgOpenFillPrice
double dCashOnHand
double dClosedPnl
double dImpliedMarginReserved
double dMarginBalance
double dOpenPnl
double dReservedBuyingPower
double dReservedMargin
double dUsedBuyingPower
// ... 32 éléments supplémentaires
```

### `PnlInfo::eAvgOpenFillPrice`

An instance of this class represents the pnl of an Account at the Account level or for an instrument.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PnlInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
PnlInfo ()
virtual ~PnlInfo ()
```
**Public Attributes**
```cpp
bool bPossOutOfOrder
double dAccountBalance
double dAvailableBuyingPower
double dAvgOpenFillPrice
double dCashOnHand
double dClosedPnl
double dImpliedMarginReserved
double dMarginBalance
double dOpenPnl
double dReservedBuyingPower
double dReservedMargin
double dUsedBuyingPower
// ... 32 éléments supplémentaires
```

### `PriceIncrInfo`

An instance of this class represents the minimum price increment of an instrument. It is possible that the minimum price increment changes based on the current price of the instrument. A PriceIncrInfo object is used to pass an array of PriceIncrRow instances. Each row indicates a minimum price increment for a range of prices. If the minimum price increment is constant, there will only be one row in the array. The PriceIncrInfo for an instrument can be requested by calling REngine::getPriceIncrInfo() ;

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PriceIncrInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
PriceIncrInfo ()
virtual ~PriceIncrInfo ()
```
**Public Attributes**
```cpp
PriceIncrRow * asPriceIncrArray
int iArrayLen
int iRpCode
tsNCharcb sExchange
tsNCharcb sRpCode
tsNCharcb sTicker
```

### `PriceIncrRow`

An instance of this class represents the minimum price increment for a price range. A PriceIncrRow object is used to indicate a minimum price increment for a range of prices. The range is defined by an upper and/or lower bound on the price. A value of OPER_NONE for an operator means that there is no constraint. When the operator is OPER_NONE, the corresponding price is not valid and should not be examined. Therefore, if both operators are OPER_NONE, there are no constraints on the price range, and the minimum price increment is constant for all prices. For example, if the minimum price increment is 0.25 for all prices, the instance will have the following values : PriceIncrRow::dPriceIncr = 0.25 PriceIncrRow::iPrecision = 2 PriceIncrRow::iFirstOperator = RApi::OP_NONE PriceIncrRow::iLastOperator = RApi::OP_NONE If one operator is OPER_NONE and the other is not, there is a single constraint. For example, if the minimum price increment is 0.10 when the price of the instrument falls below -3.00, then the instance will have the followin gvalues : PriceIncrRow::dPriceIncr = 0.10 PriceIncrRow::iPrecision = 2 PriceIncrRow::iFirstOperator = RApi::OP_LESS_THAN PriceIncrRow::dFirstPrice = -3.00 PriceIncrRow::iLastOperator = RApi::OP_NONE Another example is if the minimum price increment is 0.10 when the price of the instrument is above 3.00, the instance may have the following values : PriceIncrRow::dPriceIncr = 0.10 PriceIncrRow::iPrecision = 2 PriceIncrRow::iFirstOperator = RApi::OP_NONE PriceIncrRow::iLastOperator = RApi::OP_GREATER_THAN PriceIncrRow::dLastPrice = 3.0 Finally, if both operators are not OPER_NONE, then there are two constraints that apply to the current price increment. For example, if the minimum price increment is 0.05 when the price is greater or equal to -3.00 and less than or equal to 3.00, the instance may have the following values : PriceIncrRow::dPriceIncr = 0.05 PriceIncrRow::iPrecision = 2 PriceIncrRow::iFirstOperator = RApi::OP_GREATER_THAN_OR_EQUAL PriceIncrRow::dFirstPrice = -3.00 PriceIncrRow::iLastOperator = RApi::OP_LESS_THAN_OR_EQUAL PriceIncrRow::dLastPrice = 3.00

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PriceIncrRow.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
PriceIncrRow ()
virtual ~PriceIncrRow ()
```
**Public Attributes**
```cpp
double dFirstPrice
double dLastPrice
double dPriceIncr
int iFirstOperator
int iLastOperator
int iPrecision
```

### `ProductRmsInfo`

An instance of this class represents the product level risk management settings.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ProductRmsInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
ProductRmsInfo ()
virtual ~ProductRmsInfo ()
```
**Public Attributes**
```cpp
bool bBuyLimit
bool bBuyLongMarginRate
bool bCommissionFillRate
bool bLossLimit
bool bMaxOrderQty
bool bSellLimit
bool bSellShortMarginRate
double dBuyLongMarginRate
double dCommissionFillRate
double dLossLimit
double dSellShortMarginRate
long long llBuyLimit
// ... 4 éléments supplémentaires
```

### `ProductRmsListInfo`

An instance of this class represents a list of ProductRmsInfo instances. An ProductRmsListInfo object is used to pass an array of ProductRmsInfo via RCallbacks::ProductRmsList() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ProductRmsListInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
ProductRmsListInfo ()
virtual ~ProductRmsListInfo ()
```
**Public Attributes**
```cpp
ProductRmsInfo * asProductRmsInfoArray
int iArrayLen
int iRpCode
tsNCharcb sRpCode
```

### `ProjectedSettlementPriceInfo`

This class represents projected settlement price information. The projected settlement price is what the expected settlment price might be for an instrument. The calculation of this price is based on the settlement procedure as documented by an exchange. For example, some exchanges might base the settlement price on the VWAP for some time period (settlement period) leading up to the trading session close. This value is not available for all instruments. To subsribe for this data, the RApi::MD_PROJECTED_SETTLEMENT flag must be enabled when calling REngine::subscribe() or REngine::subscribeByUnderlying() . The callback used to convey projected settlement prices is RCallbacks::ProjectedSettlementPrice() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ProjectedSettlementPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
ProjectedSettlementPriceInfo ()
virtual ~ProjectedSettlementPriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `QuoteCancelParams`

This class encapsulates information needed to submit a cancel request for a single instrument's quote. See also REngine::cancelQuoteList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1QuoteCancelParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
QuoteCancelParams ()
virtual ~QuoteCancelParams ()
```
**Public Attributes**
```cpp
AccountInfo * pAccount
tsNCharcb sExchange
tsNCharcb sTicker
tsNCharcb sUserMsg
```

### `QuoteInfo`

This class represents information about quotes. The QuoteInfo class is used to pass quote information (including various states of the quote) via RCallbacks::Quote . See also REngine::submitQuoteList REngine::cancelQuoteList RCallbacks::Quote

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1QuoteInfo.html`

**Public Member Functions**
```cpp
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
QuoteInfo ()
virtual ~QuoteInfo ()
```
**Public Attributes**
```cpp
bool bAsk
bool bBid
double dAskPriceToFill
double dBidPriceToFill
int iSsboe
int iType
int iUsecs
long long llAskQuantityToFill
long long llBidQuantityToFill
AccountInfo oAccount
tsNCharcb sExchange
tsNCharcb sQuoteId
// ... 6 éléments supplémentaires
```

### `QuoteParams`

This class encapsulates information needed to submit a quote for a single instrument. Quotes must be placed on the ask/offer side, and can optionally be placed on the bid side. See also REngine::submitQuoteList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1QuoteParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
QuoteParams ()
virtual ~QuoteParams ()
```
**Public Attributes**
```cpp
bool bBid
double dAskPrice
double dBidPrice
int iAskQty
int iBidQty
AccountInfo * pAccount
tsNCharcb sExchange
tsNCharcb sTicker
tsNCharcb sUserMsg
```

### `QuoteReplayInfo`

This class demarcates the end of a stream of historical quote data. The QuoteReplayInfo class is used to convey the conclusion and/or response code associated with a call to REngine::replayQuotes() via RCallbacks::QuoteReplay() . Information about the actual quotes will be conveyed via RCallbacks::Quote() , with the QuoteInfo instance's callback type set to indicate historical data.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1QuoteReplayInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
QuoteReplayInfo ()
virtual ~QuoteReplayInfo ()
```
**Public Attributes**
```cpp
int iRpCode
AccountInfo oAccount
void * pContext
tsNCharcb sRpCode
```

### `QuoteReport`

The QuoteReport represents an update to a quote related request. To receive quote reports, one must subscribe to quotes. Upon receipt of an update, the quote report callback will be invoked, passing an instance of QuoteReport , conveying the update. See also REngine::subscribeOrder() REngine::submitQuoteList() REngine::cancelQuoteList() RCallbacks::Quote()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1QuoteReport.html`

**Public Member Functions**
```cpp
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
QuoteReport ()
virtual ~QuoteReport ()
```
**Public Attributes**
```cpp
bool bAsk
bool bBid
bool bFill
double dAskPriceToFill
double dBidPriceToFill
double dFillPrice
int iConnId
int iSourceNsecs
int iSourceSsboe
int iSsboe
int iType
int iUsecs
// ... 14 éléments supplémentaires
```

### `RApi::InstrumentSearchInfo`

An instance of this class marks the conclusion of the results returned from a call to REngine::searchInstrument() . REngine::searchInstrument() returns zero to many instruments, with each returned instrument being conveyed via RCallbacks::RefData() . When the results are complete, RCallbacks::InstrumentSearch() will be invoked, with this class conveying the initial parameters. See also REngine::searchInstrument() RCallbacks::RefData() RCallbacks::InstrumentSearch()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1InstrumentSearchInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
InstrumentSearchInfo ()
virtual ~InstrumentSearchInfo ()
```
**Public Attributes**
```cpp
SearchTerm * aoTerms
RefDataInfo * asArray
int iArrayLen
int iNumTerms
int iRpCode
void * pContext
tsNCharcb sExchange
```

### `RCallbacks::AskQuote()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::AuxRefData()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::BestAskQuote()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::BestBidAskQuote()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::BestBidQuote()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::BidQuote()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::CloseMidPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::ClosePrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::EndQuote()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::HighBidPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::HighPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::HighPriceLimit()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::LimitOrderBook()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::LowAskPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::LowPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::LowPriceLimit()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::MarketMode()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::MidPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::OpenInterest()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::OpenPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::OpeningIndicator()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::ProjectedSettlementPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::RefData()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::SettlementPrice()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::TradeCondition()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::TradePrint()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `RCallbacks::TradeVolume()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `REngine::replayBars()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngine::searchInstrument()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngine::subscribeBar()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `RefDataInfo`

This class represents reference data for an instrument. The RefDataInfo class is used to pass reference data received based on resulting from calls to REngine::getRefData() via the RCallbacks::RefData() callback.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RefDataInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
RefDataInfo ()
virtual ~RefDataInfo ()
```
**Public Attributes**
```cpp
bool bCapPrice
bool bFloorPrice
bool bMaxPriceVariation
bool bMinSizeIncrement
bool bSinglePointValue
bool bSizeMultiplier
bool bStrikePrice
double dCapPrice
double dFloorPrice
double dMaxPriceVariation
double dSinglePointValue
double dSizeMultiplier
// ... 24 éléments supplémentaires
```

### `ReplayBarParams`

This class conveys parameters needed to request historical data bars. Additional parameters are specified in the base class, BarParams . See also BarParams

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ReplayBarParams.html`

**Public Member Functions**
```cpp
ReplayBarParams ()
virtual ~ReplayBarParams ()
BarParams ()
virtual ~BarParams ()
```
**Public Attributes**
```cpp
int iEndSsboe
int iEndUsecs
int iStartSsboe
int iStartUsecs
tsNCharcb sEndDate
tsNCharcb sStartDate
bool bCustomSession
bool bProfile
double dSpecifiedRange
int iCloseSsm
int iOpenSsm
int iSpecifiedMinutes
// ... 7 éléments supplémentaires
```

### `SettlementPriceInfo`

This class represents settlement price information as published by an exchange. The SettlementPriceInfo class is used to present settlement price information. The method REngine::subscribe() with MD_SETTLEMENT in the subscription flags is used to express interest in receiving settlement prices. The callback used to pass that information is RCallbacks::SettlementPrice() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1SettlementPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
SettlementPriceInfo ()
virtual ~SettlementPriceInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sDate
tsNCharcb sExchange
tsNCharcb sPriceType
tsNCharcb sTicker
```

### `StrategyInfo`

This class represents the definition of a multi-leg strategy (spread). An StrategyInfo object is used to present a multi-leg spread, with the ratios of each leg. The legs are presented as an array of StrategyLegInfos. This information is retrieved using REngine::getStrategyInfo() . See also StrategyInfo StrategyListInfo REngine::getStrategyList() REngine::getStrategyInfo() RCallbacks::Strategy() RCallbacks::StrategyList()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1StrategyInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
StrategyInfo ()
virtual ~StrategyInfo ()
```
**Public Attributes**
```cpp
StrategyLegInfo * asLegArray
int iLegArrayLen
int iRpCode
void * pContext
tsNCharcb sExchange
tsNCharcb sProductCode
tsNCharcb sRpCode
tsNCharcb sTicker
tsNCharcb sType
```

### `StrategyLegInfo`

This class represents the definition of one leg of a strategy. The StrategyLegInfo class is used to pass information about one leg of a multi-leg strategy ( StrategyInfo ). See also StrategyInfo StrategyListInfo REngine::getStrategyList() REngine::getStrategyInfo() RCallbacks::Strategy() RCallbacks::StrategyList()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1StrategyLegInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
StrategyLegInfo ()
virtual ~StrategyLegInfo ()
```
**Public Attributes**
```cpp
int iRatio
tsNCharcb sExchange
tsNCharcb sInstrumentType
tsNCharcb sProductCode
tsNCharcb sTicker
```

### `StrategyListInfo`

An instance of this class is the response to a call to REngine::getStrategyList() . An StrategyListInfo object is used to pass : a list of exchanges, a list of product codes within an exchange, a list of strategy types within a product code (within an exchange), a list of expiration dates within a strategy type (within a product code/exchange), or a list of tickers representing strategies within an expiration date What is returned depends upon what was initially passed into REngine::getStrategyList() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1StrategyListInfo.html`

**Public Member Functions**
```cpp
virtual int clearHandles (int *aiCode)
int dump (int *aiCode)
StrategyListInfo ()
virtual ~StrategyListInfo ()
```
**Public Attributes**
```cpp
int iExchangeArrayLen
int iExpirationCCYYMMArrayLen
int iProductCodeArrayLen
int iRpCode
int iStrategyTypeArrayLen
int iTickerArrayLen
void * pContext
tsNCharcb * pExchangeArray
tsNCharcb * pExpirationCCYYMMArray
tsNCharcb * pProductCodeArray
tsNCharcb * pStrategyTypeArray
tsNCharcb * pTickerArray
// ... 1 éléments supplémentaires
```

### `TradeInfo`

This class represents trade information. The TradeInfo class is used to pass trade information received as a result of calls to REngine::subscribe() requests via the RCallbacks::TradePrint() callback.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1TradeInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
TradeInfo ()
virtual ~TradeInfo ()
```
**Public Attributes**
```cpp
bool bNetChangeFlag
bool bPercentChangeFlag
bool bPriceFlag
bool bVolumeBoughtFlag
bool bVolumeSoldFlag
bool bVwapFlag
bool bVwapLongFlag
double dNetChange
double dPercentChange
double dPrice
double dVwap
double dVwapLong
// ... 18 éléments supplémentaires
```

### `TradeReplayInfo`

An instance of this class represents the result of a prior call to REngine::replayTrades() . The TradeReplayInfo class is used to convey the conclusion and/or response code associated with a call to REngine::replayTrades() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1TradeReplayInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
TradeReplayInfo ()
virtual ~TradeReplayInfo ()
```
**Public Attributes**
```cpp
int iEndSsboe
int iRpCode
int iStartSsboe
tsNCharcb sExchange
tsNCharcb sRpCode
tsNCharcb sTicker
```

### `TradeRouteInfo`

This class represents a trade route for an account's fcm/ib on an exchange. The TradeRouteInfo class is used to convey the existence of an available trade route to an exchange for an fcm/ib, and to indicate its status. The trade route must be specified when placing orders. See also MarketOrderParams::sTradeRoute LimitOrderParams::sTradeRoute

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1TradeRouteInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
TradeRouteInfo ()
virtual ~TradeRouteInfo ()
```
**Public Attributes**
```cpp
int iType
void * pContext
tsNCharcb sDefault
tsNCharcb sExchange
tsNCharcb sFcmId
tsNCharcb sIbId
tsNCharcb sStatus
tsNCharcb sTradeRoute
```

### `TradeRouteListInfo`

An instance of this class represents a list of trade routes. An TradeRouteListInfo object is used to pass an array of TradeRouteInfo via RCallbacks::TradeRouteList() . See also REngine::listTradeRoutes REngine::subscribeTradeRoute REngine::unsubscribeTradeRoute RCallbacks::TradeRoute RCallbacks::TradeRouteList TradeRouteInfo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1TradeRouteListInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
TradeRouteListInfo ()
virtual ~TradeRouteListInfo ()
```
**Public Attributes**
```cpp
TradeRouteInfo * asTradeRouteInfoArray
int iArrayLen
int iRpCode
void * pContext
```

### `TradeVolumeInfo`

This class represents trade volume information. The TradeVolumeInfo class is used to pass trade volume information received as a result of calls to REngine::subscribe() requests via the RCallbacks::TradeVolume() callback.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1TradeVolumeInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
TradeVolumeInfo ()
virtual ~TradeVolumeInfo ()
```
**Public Attributes**
```cpp
bool bTotalVolumeFlag
int iJopNsecs
int iJopSsboe
int iSourceNsecs
int iSourceSsboe
int iSourceUsecs
int iSsboe
int iType
int iUsecs
long long llTotalVolume
tsNCharcb sExchange
tsNCharcb sTicker
```

### `VolumeAtPriceInfo`

This class contains the results of a request to get volume-at-price information. A VolumeAtPriceInfo object is used to present the results of a request to get volumes traded at each price level since the beginning of the trading session for a given instrument ( REngine::getVolumeAtPrice() ) See also REngine::getVolumeAtPrice() RCallbacks::VolumeAtPrice()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1VolumeAtPriceInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
VolumeAtPriceInfo ()
virtual ~VolumeAtPriceInfo ()
```
**Public Attributes**
```cpp
double * adPriceArray
long long * allVolumeArray
int iArrayLen
int iRpCode
int iSsboe
int iUsecs
void * pContext
tsNCharcb sExchange
tsNCharcb sTicker
```


---

# Trading, comptes, PnL et état

## Table des matières
- [Trading, comptes et état](#trading-comptes-et-etat)

## Trading, comptes et état

### `AccountInfo`

This class represents an account. The AccountInfo class is used to pass account information to/from RApi . The strings in the class represent a unique account on which a logged in user can trade.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AccountInfo.html`

**Public Member Functions**
```cpp
AccountInfo ()
int clearHandles (int *aiCode)
int dump (int *aiCode)
virtual ~AccountInfo ()
```
**Public Attributes**
```cpp
int iCreationSsboe
int iCreationUsecs
AutoLiquidateInfo * pAutoLiquidateInfo
RmsInfo * pRmsInfo
tsNCharcb sAccountId
tsNCharcb sAccountName
tsNCharcb sFcmId
tsNCharcb sIbId
```

### `AccountListInfo`

An instance of this class represents a list of accounts. An AccountListInfo object is used to pass an array of AccountInfo via RCallbacks::AccountList() . See also REngine::getAccounts()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AccountListInfo.html`

**Public Member Functions**
```cpp
AccountListInfo ()
int dump (int *aiCode)
virtual ~AccountListInfo ()
```
**Public Attributes**
```cpp
AccountInfo * asAccountInfoArray
int iArrayLen
int iRpCode
tsNCharcb sRpCode
```

### `AccountUpdateInfo`

An instance of this class represents a change to an account. The AccountUpdateInfo class conveys that some update has happened to the account specified by AccountUpdateInfo::oAccount . The nature of the update is indicated by what action ( AccountUpdateInfo::sAction ) has occurred. See also RCallbacks::AccountUpdate()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AccountUpdateInfo.html`

**Public Member Functions**
```cpp
AccountUpdateInfo ()
int dump (int *aiCode)
virtual ~AccountUpdateInfo ()
```
**Public Attributes**
```cpp
AccountInfo oAccount
tsNCharcb sAction
tsNCharcb sUser
```

### `AccountUpdateInfo::sAction`

An instance of this class represents a change to an account. The AccountUpdateInfo class conveys that some update has happened to the account specified by AccountUpdateInfo::oAccount . The nature of the update is indicated by what action ( AccountUpdateInfo::sAction ) has occurred. See also RCallbacks::AccountUpdate()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AccountUpdateInfo.html`

**Public Member Functions**
```cpp
AccountUpdateInfo ()
int dump (int *aiCode)
virtual ~AccountUpdateInfo ()
```
**Public Attributes**
```cpp
AccountInfo oAccount
tsNCharcb sAction
tsNCharcb sUser
```

### `AssignedUserListInfo`

This class contains a list of user ids, and represents the response resulting from a call to REngine::listAssignedUsers() . See also REngine::listAssignedUsers() RCallbacks::AssignedUserList()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AssignedUserListInfo.html`

**Public Member Functions**
```cpp
AssignedUserListInfo ()
int dump (int *aiCode)
virtual ~AssignedUserListInfo ()
```
**Public Attributes**
```cpp
int iRpCode
int iUserArrayLen
AccountInfo oAccount
void * pContext
tsNCharcb * pUserArray
```

### `BracketInfo`

This class conveys bracket-specific information aboout a bracket order. See also REngine::replayBrackets() REngine::subscribeBracket() RCallbacks::BracketReplay() RCallbacks::BracketUpdate()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BracketInfo.html`

**Public Member Functions**
```cpp
BracketInfo ()
virtual int clearHandles (int *aiCode)
int dump (int *aiCode)
virtual ~BracketInfo ()
```
**Public Attributes**
```cpp
BracketTier * asStopArray
BracketTier * asTargetArray
bool bStopsToMarketAt
bool bTargetsToMarketAfterPartialFillInSeconds
bool bTargetsToMarketAt
bool bTrailingStop
bool bTrailingStopTriggerTicks
int iStopArrayLen
int iStopsToMarketAtSsboe
int iStopsToMarketAtUsecs
int iTargetArrayLen
int iTargetsToMarketAfterPartialFillInSeconds
// ... 8 éléments supplémentaires
```

### `BracketParams`

This class conveys bracket-specific information when placing bracket orders. See also REngine::sendBracketOrder()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BracketParams.html`

**Public Member Functions**
```cpp
BracketParams ()
virtual int clearHandles (int *aiCode)
virtual ~BracketParams ()
```
**Public Attributes**
```cpp
BracketTier * asStopArray
BracketTier * asTargetArray
bool bStopsToMarketAt
bool bTargetsToMarketAfterPartialFillInSeconds
bool bTargetsToMarketAt
bool bTrailingStop
bool bTrailingStopTriggerTicks
int iStopArrayLen
int iStopsToMarketAtSsboe
int iStopsToMarketAtUsecs
int iTargetArrayLen
int iTargetsToMarketAfterPartialFillInSeconds
// ... 8 éléments supplémentaires
```

### `BracketParams::sOrderOperationType`

This class conveys bracket-specific information when placing bracket orders. See also REngine::sendBracketOrder()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BracketParams.html`

**Public Member Functions**
```cpp
BracketParams ()
virtual int clearHandles (int *aiCode)
virtual ~BracketParams ()
```
**Public Attributes**
```cpp
BracketTier * asStopArray
BracketTier * asTargetArray
bool bStopsToMarketAt
bool bTargetsToMarketAfterPartialFillInSeconds
bool bTargetsToMarketAt
bool bTrailingStop
bool bTrailingStopTriggerTicks
int iStopArrayLen
int iStopsToMarketAtSsboe
int iStopsToMarketAtUsecs
int iTargetArrayLen
int iTargetsToMarketAfterPartialFillInSeconds
// ... 8 éléments supplémentaires
```

### `BracketReplayInfo`

This class conveys the results from a call to REngine::replayBrackets() . If the request was handled without error, the iRpCode will be zero. Each bracket is represented by a BracketInfo instance. This bracket information will not provide all order details, but the bracket-specific order details. See also REngine::replayBrackets RCallbacks::BracketReplay

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BracketReplayInfo.html`

**Public Member Functions**
```cpp
BracketReplayInfo ()
int dump (int *aiCode)
virtual ~BracketReplayInfo ()
```
**Public Attributes**
```cpp
BracketInfo * asBracketArray
int iArrayLen
int iRpCode
AccountInfo oAccount
```

### `BracketTier`

This class represents one exit tier of a bracket order. See also BracketInfo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BracketTier.html`

**Public Member Functions**
```cpp
BracketTier ()
BracketTier (const int iTicksIn, const long long llQtyIn, const long long llQtyReleasedIn)
virtual ~BracketTier ()
```
**Public Attributes**
```cpp
int iTicks
long long llQty
long long llQtyReleased
```

### `BracketTierModifyInfo`

This class conveys the results of a call to REngine::modifyBracketTier() . Except for iRpCode, this class just returns the parameters passed in to REngine::modifyBracketTier() See also REngine::modifyBracketTier RCallbacks::BracketTierModify

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BracketTierModifyInfo.html`

**Public Member Functions**
```cpp
BracketTierModifyInfo ()
int dump (int *aiCode)
virtual ~BracketTierModifyInfo ()
```
**Public Attributes**
```cpp
bool bTarget
int iNewTickOffset
int iOldTickOffset
int iRpCode
AccountInfo oAccount
void * pContext
tsNCharcb sOrderNum
```

### `ExecutionReplayInfo`

An instance of this class represents a collection of fills. Each execution will be represented by a OrderFillReport instance. The ExecutionReplayInfo class is used to pass a list of executions via the RCallbacks::ExecutionReplay callback resulting from a call to REngine::replayExecutions() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ExecutionReplayInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
ExecutionReplayInfo ()
virtual ~ExecutionReplayInfo ()
```
**Public Attributes**
```cpp
OrderFillReport * asFillReportArray
int iArrayLen
int iRpCode
AccountInfo oAccount
tsNCharcb sRpCode
```

### `LimitOrderParams`

This class encapsulates information needed to submit a limit order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LimitOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
LimitOrderParams ()
virtual ~LimitOrderParams ()
```
**Public Attributes**
```cpp
double dPrice
int iQty
AccountInfo * pAccount
void * pContext
tsNCharcb sBuySellType
tsNCharcb sDuration
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sRoutingInstructions
tsNCharcb sTag
tsNCharcb sTicker
tsNCharcb sTradeRoute
// ... 2 éléments supplémentaires
```

### `LineInfo`

This class represents a cumulative view of orders. The LineInfo class is used to pass order information (including various states of the order) via RCallbacks::LineUpdate , RCallbacks::OrderReplay or RCallbacks::OpenOrderReplay . See also REngine::cancelOrder REngine::cancelAllOrders REngine::modifyOrder REngine::replayOpenOrders REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LineInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
LineInfo ()
virtual ~LineInfo ()
```
**Public Attributes**
```cpp
bool bAvgFillPriceFlag
bool bCancelAfter
bool bCancelSsboe
bool bCancelUsecs
bool bPriceToFillFlag
bool bReleaseSsboe
bool bReleaseUsecs
bool bToMarketAfterPartialFillInSeconds
bool bToMarketAt
bool bTriggerPriceFlag
double dAvgFillPrice
double dPriceToFill
// ... 65 éléments supplémentaires
```

### `MarketOrderParams`

This class encapsulates information needed to submit a market order. The only optional value is MarketOrderParams::pContext .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1MarketOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
MarketOrderParams ()
virtual ~MarketOrderParams ()
```
**Public Attributes**
```cpp
int iQty
AccountInfo * pAccount
void * pContext
tsNCharcb sBuySellType
tsNCharcb sDuration
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sRoutingInstructions
tsNCharcb sTag
tsNCharcb sTicker
tsNCharcb sTradeRoute
tsNCharcb sTradingAlgorithm
// ... 1 éléments supplémentaires
```

### `ModifyLimitOrderParams`

This class encapsulates information needed to modify a limit order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ModifyLimitOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
ModifyLimitOrderParams ()
virtual ~ModifyLimitOrderParams ()
```
**Public Attributes**
```cpp
double dPrice
int iQty
AccountInfo * pAccount
void * pContext
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sOrderNum
tsNCharcb sTicker
tsNCharcb sTradingAlgorithm
tsNCharcb sUserMsg
```

### `ModifyOrderParams`

This class encapsulates information needed to modify an order. If a trigger price is specified and the order type is MIT/LIT, then the trigger exchange and trigger ticker must be specified as well. The trigger price will be assumed to be a price for the trigger exch/ticker. If a trigger price is specified and the order type is not MIT/LIT, then the trigger price is assumed to be a price for the exchange and ticker. Changing other parameters of MIT/LIT orders is not supported. In these cases, the recommendation is to cancel and replace the MIT/LIT order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ModifyOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
ModifyOrderParams ()
ModifyOrderParams (ModifyLimitOrderParams *pParamsIn)
ModifyOrderParams (ModifyStopLimitOrderParams *pParamsIn)
ModifyOrderParams (ModifyStopMarketOrderParams *pParamsIn)
virtual ~ModifyOrderParams ()
```
**Public Attributes**
```cpp
bool bMinQty
bool bPrice
bool bQty
bool bTrailByTicks
bool bTriggerPrice
double dPrice
double dTriggerPrice
int iMinQty
int iQty
int iTrailByTicks
AccountInfo * pAccount
void * pContext
// ... 10 éléments supplémentaires
```

### `ModifyStopLimitOrderParams`

This class encapsulates information needed to modify a stop limit order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ModifyStopLimitOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
ModifyStopLimitOrderParams ()
virtual ~ModifyStopLimitOrderParams ()
```
**Public Attributes**
```cpp
double dPrice
double dTriggerPrice
int iQty
AccountInfo * pAccount
void * pContext
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sOrderNum
tsNCharcb sTicker
tsNCharcb sTradingAlgorithm
tsNCharcb sUserMsg
```

### `ModifyStopMarketOrderParams`

This class encapsulates information needed to modify a stop limit order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ModifyStopMarketOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
ModifyStopMarketOrderParams ()
virtual ~ModifyStopMarketOrderParams ()
```
**Public Attributes**
```cpp
double dTriggerPrice
int iQty
AccountInfo * pAccount
void * pContext
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sOrderNum
tsNCharcb sTicker
tsNCharcb sTradingAlgorithm
tsNCharcb sUserMsg
```

### `OrderBustReport`

The OrderBustReport represents a trade which has been busted and is passed in RCallbacks::BustReport() . The OrderBustReport class conveys information about a trade bust. See also REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderBustReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderBustReport ()
virtual ~OrderBustReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
tsNCharcb sFillType
tsNCharcb sSettlementCurrency
tsNCharcb sSettlementDate
tsNCharcb sTradeDate
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
// ... 41 éléments supplémentaires
```

### `OrderCancelReport`

The OrderCancelReport represents a cancellation of an order and is passed in RCallbacks::CancelReport() . The OrderCancelReport class conveys information about a cancellation of a full or partial order, depending on what portion of the order could be cancelled. See also REngine::cancelOrder REngine::cancelAllOrders

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderCancelReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderCancelReport ()
virtual ~OrderCancelReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
long long llCancelledSize
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
// ... 38 éléments supplémentaires
```

### `OrderFailureReport`

The OrderFailureReport represents a failure of an order operation to be placed at the exchange and is passed in RCallbacks::FailureReport() . The OrderFailureReport callback is invoked when an order operation cannot be placed with the exchange. These operations include the placement of new orders and the cancellation or modification of existing orders. See also RCallbacks::FailureReport REngine::cancelAllOrders REngine::cancelOrder REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderFailureReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderFailureReport ()
virtual ~OrderFailureReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
long long llCancelledSize
tsNCharcb sStatus
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
// ... 39 éléments supplémentaires
```

### `OrderFillReport`

The OrderFillReport represents a fill for an order and is passed in RCallbacks::FillReport() . The OrderFillReport class conveys non-cumulative information about a full or partial fill for an order. A fill report for a strategy (an instrument with multiple components like a calendar spread) will be followed by fill reports for each of the legs. The aggregate of these leg fill reports is equivalent to the fill report of the strategy. If price reference data is missing for any of these leg instruments, it is possible that the fill report callback for that leg instrument will not be invoked. See also REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderFillReport.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
OrderFillReport ()
virtual ~OrderFillReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
int iRpCode
tsNCharcb sFillType
tsNCharcb sLiquidityIndicator
tsNCharcb sQuoteId
tsNCharcb sQuoteMsgId
tsNCharcb sRpCode
tsNCharcb sSettlementCurrency
tsNCharcb sSettlementDate
tsNCharcb sTradeDate
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
// ... 46 éléments supplémentaires
```

### `OrderHistoryDatesInfo`

An instance of this class represents the dates in which order history is available. An OrderHistoryDatesInfo object is used to pass an array of tsNCharcbs via RCallbacks::OrderHistoryDates() .

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderHistoryDatesInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderHistoryDatesInfo ()
virtual ~OrderHistoryDatesInfo ()
```
**Public Attributes**
```cpp
tsNCharcb * asDateArray
int iArrayLen
int iRpCode
void * pContext
tsNCharcb sRpCode
```

### `OrderModifyReport`

The OrderModifyReport represents the successful modification of an existing order and is passed in RCallbacks::ModifyReport() . The OrderModifyReport class conveys information about an order as it stands after a successful modification. See also REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderModifyReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderModifyReport ()
virtual ~OrderModifyReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
long long llNewSize
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
// ... 38 éléments supplémentaires
```

### `OrderNotCancelledReport`

The OrderNotCancelledReport represents the failure of a request to cancel an existing order and is passed in RCallbacks::NotCancelledReport() . The OrderNotCancelledReport class conveys information about the failure to cancel an order. See also REngine::cancelOrder REngine::cancelAllOrders

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderNotCancelledReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderNotCancelledReport ()
virtual ~OrderNotCancelledReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
long long llNotCancelledSize
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
// ... 38 éléments supplémentaires
```

### `OrderNotModifiedReport`

The OrderNotModifiedReport represents the failure of a request to modify an existing order and is passed in RCallbacks::NotModifiedReport() . The OrderNotModifiedReport class conveys information about the failure to modify an order. See also REngine::modifyOrder

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderNotModifiedReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderNotModifiedReport ()
virtual ~OrderNotModifiedReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
long long llNotModifiedSize
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
// ... 38 éléments supplémentaires
```

### `OrderParams`

This class encapsulates information needed to submit an order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
OrderParams ()
OrderParams (LimitOrderParams *pParamsIn)
OrderParams (MarketOrderParams *pParamsIn)
OrderParams (StopLimitOrderParams *pParamsIn)
OrderParams (StopMarketOrderParams *pParamsIn)
virtual ~OrderParams ()
```
**Public Attributes**
```cpp
bool bCancelAfter
bool bCancelSsboe
bool bCancelUsecs
bool bMaxShowQty
bool bMinQty
bool bReleaseCondition
bool bReleaseSsboe
bool bReleaseUsecs
bool bToMarketAfterPartialFillInSeconds
bool bToMarketAt
bool bTrailingStop
double dPrice
// ... 41 éléments supplémentaires
```

### `OrderRejectReport`

The OrderRejectReport represents the rejection of a request to place an order and is passed in RCallbacks::RejectReport() . The OrderRejectReport class conveys information about the rejection. See also REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderRejectReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderRejectReport ()
virtual ~OrderRejectReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
bool bReplacementOrderToFollow
long long llRejectedSize
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
// ... 39 éléments supplémentaires
```

### `OrderReplayInfo`

An instance of this class represents a collection of orders. Each order will be represented by an LineInfo instance. The OrderReplayInfo class is used to pass a collection of orders via RCallbacks::OrderReplay() or RCallbacks::OpenOrderReplay() . See also REngine::replayAllOrders() REngine::replayHistoricalOrders() REngine::replayOpenOrders() RCallbacks::OpenOrderReplay() RCallbacks::OrderReplay()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderReplayInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderReplayInfo ()
virtual ~OrderReplayInfo ()
```
**Public Attributes**
```cpp
LineInfo * asLineInfoArray
int iArrayLen
int iRpCode
AccountInfo oAccount
tsNCharcb sDate
tsNCharcb sRpCode
```

### `OrderReport`

The OrderReport represents basic information about an order. The OrderReport class is the base class for other Order*Report classes, It is passed in RCallbacks::OrderOtherReport().

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderReport.html`

**Public Member Functions**
```cpp
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
int iGatewayUsecs
// ... 37 éléments supplémentaires
```

### `OrderStatusReport`

The OrderStatusReport represents the successful placement of an order and is passed in RCallbacks::StatusReport() . The OrderStatusReport class conveys information about the new order. See also REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderStatusReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderStatusReport ()
virtual ~OrderStatusReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
long long llConfirmedSize
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
// ... 38 éléments supplémentaires
```

### `OrderTriggerPulledReport`

The OrderTriggerPulledReport represents the successful placement of an order by the order release manager and is passed in RCallbacks::TriggerPulledReport() . The OrderTriggerPulledReport class conveys information about the order being placed by the order release manager. Typically, this report is only used when certain 'trigger' conditions are met, causing the placement of the order at the exchange.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderTriggerPulledReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderTriggerPulledReport ()
virtual ~OrderTriggerPulledReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Additional Inherited Members**
```cpp
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
int iGatewayUsecs
// ... 37 éléments supplémentaires
```

### `OrderTriggerReport`

The OrderTriggerReport represents the successful placement of an order and is passed in RCallbacks::TriggerReport() . The OrderTriggerReport class conveys information about the new order. See also REngine::modifyOrder REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderTriggerReport.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
OrderTriggerReport ()
virtual ~OrderTriggerReport ()
virtual int clearHandles (int *aiCode)
virtual int dump (int *aiCode)
OrderReport ()
virtual ~OrderReport ()
```
**Public Attributes**
```cpp
long long llConfirmedSize
bool bAvgFillPriceFlag
bool bFillPriceFlag
bool bPriceToFillFlag
bool bTriggerPriceFlag
double dAvgFillPrice
double dFillPrice
double dPriceToFill
double dTriggerPrice
int iConnId
int iGatewayBufferBytesBehind
int iGatewaySsboe
// ... 38 éléments supplémentaires
```

### `PasswordChangeInfo`

An instance of this class represents the outcome of an attempt to change a password.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PasswordChangeInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
PasswordChangeInfo ()
virtual ~PasswordChangeInfo ()
```
**Public Attributes**
```cpp
int iRpCode
tsNCharcb sRpCode
```

### `PnlInfo`

An instance of this class represents the pnl of an Account at the Account level or for an instrument.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PnlInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
PnlInfo ()
virtual ~PnlInfo ()
```
**Public Attributes**
```cpp
bool bPossOutOfOrder
double dAccountBalance
double dAvailableBuyingPower
double dAvgOpenFillPrice
double dCashOnHand
double dClosedPnl
double dImpliedMarginReserved
double dMarginBalance
double dOpenPnl
double dReservedBuyingPower
double dReservedMargin
double dUsedBuyingPower
// ... 32 éléments supplémentaires
```

### `PnlReplayInfo`

An instance of this class represents the p/l of an Account. A PnlReplayInfo object is used to pass an array of PnlInfo instances returned via RCallbacks::PnlReplay() resulting from a call to REngine::replayPnls().

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PnlReplayInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
PnlReplayInfo ()
virtual ~PnlReplayInfo ()
```
**Public Attributes**
```cpp
PnlInfo * asPnlInfoArray
int iArrayLen
int iRpCode
AccountInfo oAccount
tsNCharcb sRpCode
```

### `PositionExitInfo`

An instance of this class returns the result returned by the infrastructure of a call to REngine::exitPosition() . The PositionExitInfo object will have the parameters passed into REngine::exitPosition() as well as the response code for the exit position request from the infrastructure. A non-zero value of the response code iRpCode indicates some error condition. See also REngine::exitPosition()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1PositionExitInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
PositionExitInfo ()
virtual ~PositionExitInfo ()
```
**Public Attributes**
```cpp
int iRpCode
AccountInfo oAccount
void * pContext
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sTicker
tsNCharcb sTradingAlgorithm
```

### `RApi::LineInfo::sReleaseCommand`

This class represents a cumulative view of orders. The LineInfo class is used to pass order information (including various states of the order) via RCallbacks::LineUpdate , RCallbacks::OrderReplay or RCallbacks::OpenOrderReplay . See also REngine::cancelOrder REngine::cancelAllOrders REngine::modifyOrder REngine::replayOpenOrders REngine::sendOrder REngine::sendOrderList

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1LineInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
LineInfo ()
virtual ~LineInfo ()
```
**Public Attributes**
```cpp
bool bAvgFillPriceFlag
bool bCancelAfter
bool bCancelSsboe
bool bCancelUsecs
bool bPriceToFillFlag
bool bReleaseSsboe
bool bReleaseUsecs
bool bToMarketAfterPartialFillInSeconds
bool bToMarketAt
bool bTriggerPriceFlag
double dAvgFillPrice
double dPriceToFill
// ... 65 éléments supplémentaires
```

### `RApi::OrderParams::sReleaseCommand`

This class encapsulates information needed to submit an order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1OrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
OrderParams ()
OrderParams (LimitOrderParams *pParamsIn)
OrderParams (MarketOrderParams *pParamsIn)
OrderParams (StopLimitOrderParams *pParamsIn)
OrderParams (StopMarketOrderParams *pParamsIn)
virtual ~OrderParams ()
```
**Public Attributes**
```cpp
bool bCancelAfter
bool bCancelSsboe
bool bCancelUsecs
bool bMaxShowQty
bool bMinQty
bool bReleaseCondition
bool bReleaseSsboe
bool bReleaseUsecs
bool bToMarketAfterPartialFillInSeconds
bool bToMarketAt
bool bTrailingStop
double dPrice
// ... 41 éléments supplémentaires
```

### `RCallbacks::PnlUpdate`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `REngine::getAccounts()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `RmsInfo`

This class represents the risk management settings for an account. The RmsInfo class is used to convey risk parameters for an account to a user of the RApi . The account balance (see PnlInfo::dAccountBalance ) is calculated by summing the open PnL and closed PnL (see PnlInfo::dOpenPnl and PnlInfo::dClosedPnl ), subtracting the total commission, and adding the cash on hand. The total commission is calculated by multiplying the commission fill rate by the quantity filled for all orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RmsInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
RmsInfo ()
virtual ~RmsInfo ()
```
**Public Attributes**
```cpp
bool bAutoLiquidateAtSsm
bool bAutoLiquidateMaxMinAccountBalance
bool bAutoLiquidateThreshold
bool bCheckMinAccountBalance
bool bDefaultCommission
bool bMinMarginBalance
double dAutoLiquidateMaxMinAccountBalance
double dAutoLiquidateThreshold
double dDefaultCommission
double dLossLimit
double dMinAccountBalance
double dMinMarginBalance
// ... 11 éléments supplémentaires
```

### `RmsInfo::sStatus`

This class represents the risk management settings for an account. The RmsInfo class is used to convey risk parameters for an account to a user of the RApi . The account balance (see PnlInfo::dAccountBalance ) is calculated by summing the open PnL and closed PnL (see PnlInfo::dOpenPnl and PnlInfo::dClosedPnl ), subtracting the total commission, and adding the cash on hand. The total commission is calculated by multiplying the commission fill rate by the quantity filled for all orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RmsInfo.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
RmsInfo ()
virtual ~RmsInfo ()
```
**Public Attributes**
```cpp
bool bAutoLiquidateAtSsm
bool bAutoLiquidateMaxMinAccountBalance
bool bAutoLiquidateThreshold
bool bCheckMinAccountBalance
bool bDefaultCommission
bool bMinMarginBalance
double dAutoLiquidateMaxMinAccountBalance
double dAutoLiquidateThreshold
double dDefaultCommission
double dLossLimit
double dMinAccountBalance
double dMinMarginBalance
// ... 11 éléments supplémentaires
```

### `SingleOrderReplayInfo`

An instance of this class represents the conclusion of results from a prior call to REngine::replaySingleOrder() . The SingleOrderReplayInfo class is used to convey the conclusion and response code associated with a call to REngine::replaySingleOrder() . See also REngine::replaySingleOrder RCallbacks::SingleOrderReplay

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1SingleOrderReplayInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
SingleOrderReplayInfo ()
virtual ~SingleOrderReplayInfo ()
```
**Public Attributes**
```cpp
int iRpCode
AccountInfo oAccount
void * pContext
tsNCharcb sDate
tsNCharcb sOrderNum
tsNCharcb sRpCode
```

### `SodReport`

An instance of this class represents a position and close price of an instrument when the trading day has ended. It is possible that different instruments on the same exchange have different ending times for a given trading day.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1SodReport.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
int dump (int *aiCode)
SodReport ()
virtual ~SodReport ()
```
**Public Attributes**
```cpp
double dPrevClosePrice
int iSsboe
long long llCarriedSize
AccountInfo oAccount
tsNCharcb sExchange
tsNCharcb sReportType
tsNCharcb sTicker
```

### `StopLimitOrderParams`

This class encapsulates information needed to submit a stop limit order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1StopLimitOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
StopLimitOrderParams ()
virtual ~StopLimitOrderParams ()
LimitOrderParams ()
virtual ~LimitOrderParams ()
```
**Public Attributes**
```cpp
double dTriggerPrice
double dPrice
int iQty
AccountInfo * pAccount
void * pContext
tsNCharcb sBuySellType
tsNCharcb sDuration
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sRoutingInstructions
tsNCharcb sTag
tsNCharcb sTicker
// ... 3 éléments supplémentaires
```

### `StopMarketOrderParams`

This class encapsulates information needed to submit a stop limit order.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1StopMarketOrderParams.html`

**Public Member Functions**
```cpp
int clearHandles (int *aiCode)
StopMarketOrderParams ()
virtual ~StopMarketOrderParams ()
MarketOrderParams ()
virtual ~MarketOrderParams ()
```
**Public Attributes**
```cpp
double dTriggerPrice
int iQty
AccountInfo * pAccount
void * pContext
tsNCharcb sBuySellType
tsNCharcb sDuration
tsNCharcb sEntryType
tsNCharcb sExchange
tsNCharcb sRoutingInstructions
tsNCharcb sTag
tsNCharcb sTicker
tsNCharcb sTradeRoute
// ... 2 éléments supplémentaires
```

### `UserDefinedSpreadCreateInfo`

An instance of this class returns the result returned by the infrastructure of a call to REngine::createUserDefinedSpread() . A non-zero value of the response code iRpCode indicates some error condition. See also REngine::createUserDefinedSpread() RCallbacks::UserDefinedSpreadCreate()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1UserDefinedSpreadCreateInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
UserDefinedSpreadCreateInfo ()
virtual ~UserDefinedSpreadCreateInfo ()
```
**Public Attributes**
```cpp
int iRpCode
void * pContext
tsNCharcb sText
tsNCharcb sTicker
```

### `UserInfo`

This class conveys information about a user. The UserInfo class conveys information about a user. See also REngine::listUsers() REngine::subscribeUser() REngine::unsubscribeUser() RCallbacks::User()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1UserInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
UserInfo ()
virtual ~UserInfo ()
```
**Public Attributes**
```cpp
int iCallbackType
int iCreationSsboe
int iCreationUsecs
int iDisableSsboe
int iFirstLoginSsboe
int iFirstLoginUsecs
int iLastLoginSsboe
int iLastLoginUsecs
int iSsboe
int iUsecs
void * pContext
tsNCharcb sAction
// ... 16 éléments supplémentaires
```

### `UserInfo::sAction`

This class conveys information about a user. The UserInfo class conveys information about a user. See also REngine::listUsers() REngine::subscribeUser() REngine::unsubscribeUser() RCallbacks::User()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1UserInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
UserInfo ()
virtual ~UserInfo ()
```
**Public Attributes**
```cpp
int iCallbackType
int iCreationSsboe
int iCreationUsecs
int iDisableSsboe
int iFirstLoginSsboe
int iFirstLoginUsecs
int iLastLoginSsboe
int iLastLoginUsecs
int iSsboe
int iUsecs
void * pContext
tsNCharcb sAction
// ... 16 éléments supplémentaires
```

### `UserInfo::sStatus`

This class conveys information about a user. The UserInfo class conveys information about a user. See also REngine::listUsers() REngine::subscribeUser() REngine::unsubscribeUser() RCallbacks::User()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1UserInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
UserInfo ()
virtual ~UserInfo ()
```
**Public Attributes**
```cpp
int iCallbackType
int iCreationSsboe
int iCreationUsecs
int iDisableSsboe
int iFirstLoginSsboe
int iFirstLoginUsecs
int iLastLoginSsboe
int iLastLoginUsecs
int iSsboe
int iUsecs
void * pContext
tsNCharcb sAction
// ... 16 éléments supplémentaires
```

### `UserListInfo`

An instance of this class represents the end of a list of users. After all of the users returned from a call to REngine::listUsers() , an instance of this class will be returned via RCallbacks::UserList() . This class indicates an end to the list of users, as well as conveying any error that may have occurred. See also REngine::listUsers RCallbacks::User RCallbacks::UserList UserInfo UserListInfo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1UserListInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
UserListInfo ()
virtual ~UserListInfo ()
```
**Public Attributes**
```cpp
int iRpCode
void * pContext
tsNCharcb sIbId
tsNCharcb sUserType
```

### `UserProfileInfo`

This class represents a user profile. The UserProfileInfo class conveys information about a user, including session information. The session information is divided by connection as well as by current/peak/max session count. When a user is logged in to a Rithmic sub-system, that is considered a current login session. As a user logs in to a particular sub-system multiple times (perhaps with different apps), the session count goes up. The peak session count is the highest number of concurrent sessions recorded by the infrastructure. The max session count is the maximum number of concurrent sessions permitted for this user id. See also REngine::getUserProfile() RCallbacks::UserProfile()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1UserProfileInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
UserProfileInfo ()
virtual ~UserProfileInfo ()
```
**Public Attributes**
```cpp
int iConnId
int iCurrentHistorySessionCount
int iCurrentMarketDataSessionCount
int iCurrentOrdersSessionCount
int iCurrentPnlSessionCount
int iCurrentRepositorySessionCount
int iMaxHistorySessionCount
int iMaxMarketDataSessionCount
int iMaxOrdersSessionCount
int iMaxPnlSessionCount
int iMaxRepositorySessionCount
int iPeakHistorySessionCount
// ... 21 éléments supplémentaires
```


---

# Administration, recherche, reprise et rejeu

## Table des matières
- [Administration, recherche et rejeu](#administration-recherche-et-rejeu)
- [Autres structures](#autres-structures)

## Administration, recherche et rejeu

### `AgreementInfo`

This class represents an agreement. The AgreementInfo class is used to convey agreement meta-data. One can request agreement information by calling REngine::listAgreements() . Please read the FAQ on agreements for more information : FAQ - Agreements See also REngine::listAgreements RCallbacks::AgreementList AgreementListInfo AgreementInfo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AgreementInfo.html`

**Public Member Functions**
```cpp
AgreementInfo ()
int dump (int *aiCode)
virtual ~AgreementInfo ()
```
**Public Attributes**
```cpp
bool bMandatory
int iEndUserAcceptanceSsboe
int iInForceSsboe
int iPublicationSsboe
tsNCharcb sEndUserAcceptanceStatus
tsNCharcb sMarketDataUsageCapacity
tsNCharcb sStatus
tsNCharcb sTitle
```

### `AgreementListInfo`

An instance of this class represents a collection of agreements. An AgreementListInfo object is passed via RCallbacks::AgreementList() , and conveys the results of a call to REngine::listAgreements() . It returns the initial parameters, a response code indicating whether or not an error has occurred, and, if applicable, an array of AgreementInfo instances. Please read the FAQ on agreements for more information : FAQ - Agreements See also REngine::listAgreements RCallbacks::AgreementList AgreementListInfo AgreementInfo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AgreementListInfo.html`

**Public Member Functions**
```cpp
AgreementListInfo ()
int dump (int *aiCode)
virtual ~AgreementListInfo ()
```
**Public Attributes**
```cpp
AgreementInfo * asAgreementInfoArray
bool bAccepted
int iArrayLen
int iRpCode
void * pContext
tsNCharcb sRpCode
```

### `AlertInfo`

This class represents a system alert. The AlertInfo class is used to pass alert information via RCallbacks::Alert() . Alerts typically convey information about the system infrastructure, rather than data content. It is possible that one or more of members of AlertInfo may not be populated.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AlertInfo.html`

**Public Member Functions**
```cpp
AlertInfo ()
int dump (int *aiCode)
virtual ~AlertInfo ()
```
**Public Attributes**
```cpp
int iAlertType
int iConnectionId
int iRpCode
tsNCharcb sExchange
tsNCharcb sMessage
tsNCharcb sRpCode
tsNCharcb sTicker
```

### `EnvironmentInfo`

List of environment variables in a given environment. An EnvironmentInfo object conveys the list of environment variables comprising an environment. See also REngine::getEnvironment() AdmCallbacks::Environment() EnvironmentVariable

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1EnvironmentInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
EnvironmentInfo ()
virtual ~EnvironmentInfo ()
```
**Public Attributes**
```cpp
EnvironmentVariable * asVariableArray
int iArrayLen
void * pContext
tsNCharcb sKey
```

### `EnvironmentListInfo`

List of environment keys. An EnvironmentListInfo object is used to pass an array of environment keys. See also REngine::listEnvironments() AdmCallbacks::EnvironmentList()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1EnvironmentListInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
EnvironmentListInfo ()
virtual ~EnvironmentListInfo ()
```
**Public Attributes**
```cpp
tsNCharcb * asKeyArray
int iArrayLen
void * pContext
```

### `EnvironmentVariable`

Environment variable's name-value pair. An EnvironmentVariable object is a name-value pair of an environment. See also REngine::getEnvironment() AdmCallbacks::Environment() EnvironmentInfo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1EnvironmentVariable.html`

**Public Member Functions**
```cpp
EnvironmentVariable ()
virtual ~EnvironmentVariable ()
```
**Public Attributes**
```cpp
tsNCharcb sName
tsNCharcb sValue
```

### `RApi::SearchTerm`

This class represents a search term for REngine::searchInstrument() . One or more instances of the SearchTerm class is passed into REngine::searchInstrument() to specify a filter when searching for instruments.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1SearchTerm.html`

**Public Attributes**
```cpp
bool bCaseSensitive
SearchField eField
SearchOperator eOperator
tsNCharcb sTerm
```

### `SearchTerm::bCaseSensitive`

This class represents a search term for REngine::searchInstrument() . One or more instances of the SearchTerm class is passed into REngine::searchInstrument() to specify a filter when searching for instruments.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1SearchTerm.html`

**Public Attributes**
```cpp
bool bCaseSensitive
SearchField eField
SearchOperator eOperator
tsNCharcb sTerm
```

### `SearchTerm::sTerm`

This class represents a search term for REngine::searchInstrument() . One or more instances of the SearchTerm class is passed into REngine::searchInstrument() to specify a filter when searching for instruments.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1SearchTerm.html`

**Public Attributes**
```cpp
bool bCaseSensitive
SearchField eField
SearchOperator eOperator
tsNCharcb sTerm
```

## Autres structures

### `AggregatorInfo`

Returns information about whether the market data connection is to an aggregated ticker plant. See also REngine::isThereAnAggregator() RCallbacks::Aggregator()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AggregatorInfo.html`

**Public Member Functions**
```cpp
AggregatorInfo ()
int dump (int *aiCode)
virtual ~AggregatorInfo ()
```
**Public Attributes**
```cpp
int iRpCode
```

### `AutoLiquidateInfo`

An instance of this class represents the state of an Account's auto-liquidate threshold (current) value, along with the time of that value. See also REngine::subscribeAutoLiquidate() REngine::unsubscribeAutoLiquidate() RCallbacks::AutoLiquidate()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1AutoLiquidateInfo.html`

**Public Member Functions**
```cpp
AutoLiquidateInfo ()
virtual int clearHandles (int *aiCode)
int dump (int *aiCode)
virtual ~AutoLiquidateInfo ()
```
**Public Attributes**
```cpp
double dAutoLiquidateThresholdCurrentValue
int eAutoLiquidateThresholdCurrentValue
int eAutoLiquidateThresholdCurrentValueSsboe
int iAutoLiquidateThresholdCurrentValueSsboe
int iCallbackType
AccountInfo oAccount
```

### `BinaryContractListInfo`

An instance of this class is the response to a call to REngine::listBinaryContracts() . An BinaryContractListInfo object is used to pass : a list of exchanges, a list of product codes within an exchange, a list of binary contract types within that product code, a list of period codes within that binary contract type, a list of expiration times within that period code, or a list of RefDataInfo objects representing binary contracts with the particular features of exchange, product code, binary contract type, period code and expiration time. What is returned depends upon what was initially passed into REngine::listBinaryContracts() . See also REngine::listBinaryContracts() RCallbacks::BinaryContractList()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1BinaryContractListInfo.html`

**Public Member Functions**
```cpp
BinaryContractListInfo ()
virtual int clearHandles (int *aiCode)
int dump (int *aiCode)
virtual ~BinaryContractListInfo ()
```
**Public Attributes**
```cpp
RefDataInfo * asRefDataInfoArray
int iArrayLen
int iBinaryContractTypeArrayLen
int iExchangeArrayLen
int iExpirationTimeArrayLen
int iPeriodCodeArrayLen
int iProductCodeArrayLen
int iRpCode
tsNCharcb * pBinaryContractTypeArray
void * pContext
tsNCharcb * pExchangeArray
tsNCharcb * pExpirationTimeArray
// ... 2 éléments supplémentaires
```

### `ClosingIndicatorInfo`

This class represents closing indicators. The ClosingIndicatorInfo class is used to pass closing indicator price information received as a result of calls to REngine::subscribe() requests via the RCallbacks::ClosingIndicator() callback. Not all exchanges or instruments make use of closing indicators.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1ClosingIndicatorInfo.html`

**Public Member Functions**
```cpp
ClosingIndicatorInfo ()
int dump (int *aiCode)
virtual ~ClosingIndicatorInfo ()
```
**Public Attributes**
```cpp
bool bPriceFlag
double dPrice
int iSsboe
int iType
int iUsecs
tsNCharcb sExchange
tsNCharcb sTicker
```

### `EasyToBorrowInfo`

Instrument's easy-to-borrow information. The EasyToBorrowInfo class is used to convey information about shares of an instrument, and if they can be borrowed. Normally this information is used when short selling equities. See also REngine::getEasyToBorrowList() REngine::subscribeEasyToBorrow() REngine::unsubscribeEasyToBorrow() RCallbacks::EasyToBorrow() RCallbacks::EasyToBorrowList() EasyToBorrowListInfo

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1EasyToBorrowInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
EasyToBorrowInfo ()
virtual ~EasyToBorrowInfo ()
```
**Public Attributes**
```cpp
bool bBorrowable
int iType
long long llQtyAvailable
long long llQtyNeeded
void * pContext
tsNCharcb sBrokerDealer
tsNCharcb sDescription
tsNCharcb sTicker
```

### `EasyToBorrowListInfo`

This class represents the status of a request to get the easy-to-borrow list. The EasyToBorrowListInfo class represents the status of a request to get the easy-to-borrow list from the infrastructure. See also REngine::getEasyToBorrowList() RCallbacks::EasyToBorrowList()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1EasyToBorrowListInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
EasyToBorrowListInfo ()
virtual ~EasyToBorrowListInfo ()
```
**Public Attributes**
```cpp
int iRpCode
void * pContext
```

### `IbListInfo`

This class contains a list of IBs, and represents the response resulting from a call to REngine::listIbs() . See also REngine::listIbs() RCallbacks::IbList()

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1IbListInfo.html`

**Public Member Functions**
```cpp
int dump (int *aiCode)
IbListInfo ()
virtual ~IbListInfo ()
```
**Public Attributes**
```cpp
int iIbIdArrayLen
int iRpCode
void * pContext
tsNCharcb * pIbIdArray
```

### `RCallbacks::ClosingIndicator()`

This class contains all the user-defined callbacks for RApi . A subclass of RCallbacks must be defined, instantiated and passed via LoginParams::pCallbacks when logging into the infrastructure. While the session is active, the instance referenced by LoginParams::pCallbacks must not be destroyed.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1RCallbacks.html`

**Public Member Functions**
```cpp
virtual int AccountList (AccountListInfo *pInfo, void *pContext, int *aiCode)
virtual int AccountUpdate (AccountUpdateInfo *pInfo, void *pContext, int *aiCode)
virtual int Aggregator (AggregatorInfo *pInfo, void *pContext, int *aiCode)
virtual int AgreementList (AgreementListInfo *pInfo, void *pContext, int *aiCode)
virtual int Alert (AlertInfo *pInfo, void *pContext, int *aiCode)
virtual int AskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
virtual int AssignedUserList (AssignedUserListInfo *pInfo, void *pContext, int *aiCode)
virtual int AutoLiquidate (AutoLiquidateInfo *pInfo, void *pContext, int *aiCode)
virtual int AuxRefData (AuxRefDataInfo *pInfo, void *pContext, int *aiCode)
virtual int Bar (BarInfo *pInfo, void *pContext, int *aiCode)
virtual int BarReplay (BarReplayInfo *pInfo, void *pContext, int *aiCode)
virtual int BestAskQuote (AskInfo *pInfo, void *pContext, int *aiCode)
// ... 79 éléments supplémentaires
```

### `REngine::login()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngine::logout`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngine::logout()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngine::sendOcaList`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngine::subscribe()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```

### `REngine::subscribeByUnderlying()`

This class initiates all functionality within RApi . The REngine class is used to connect to the infrastructure, retrieve market data, reference data and manage orders.

- Include : `#include <RApiPlus.h>`
- Page source Doxygen : `classRApi_1_1REngine.html`

**Public Member Functions**
```cpp
int addTimer (tsNCharcb *const pTimerName, const OmneEngineSpace::OmneTimerCB *pTimerCB, long lFreq, void *pContext, int *aiCode)
int cancelAllOrders (AccountInfo *pAccount, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, int *aiCode)
int cancelOrder (AccountInfo *pAccount, tsNCharcb *pOrderNum, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, tsNCharcb *pUserMsg, void *pContext, int *aiCode)
int cancelOrderList (AccountInfo *aoAccountArray, tsNCharcb *asOrderNumArray, tsNCharcb *asEntryTypeArray, tsNCharcb *asTradingAlgorithmArray, tsNCharcb *asUserMsgArray, void **apContextArray, int iArrayLen, int *aiCode)
int cancelQuoteList (QuoteCancelParams *pParams, int iArrayLen, int *aiCode)
int changePassword (tsNCharcb *pOldPassword, tsNCharcb *pNewPassword, int *aiCode)
int createUserDefinedSpread (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTradeRoute, tsNCharcb *pEntryType, tsNCharcb *pStrategyType, int iNumLegs, tsNCharcb *asLegTickers, int *aiLegRatios, void *pContext, int *aiCode)
int exitPosition (AccountInfo *pAccount, tsNCharcb *pExchange, tsNCharcb *pTicker, tsNCharcb *pEntryType, tsNCharcb *pTradingAlgorithm, void *pContext, int *aiCode)
int getAccounts (tsNCharcb *pStatus, int *aiCode)
int getAdmCallbacks (AdmCallbacks **apCallbacks, int *aiCode)
int getAdminName (tsNCharcb *pAdminName, int *aiCode)
int getAppName (tsNCharcb *pAppName, int *aiCode)
// ... 106 éléments supplémentaires
```
**Static Public Member Functions**
```cpp
static int getVersion (tsNCharcb *pVersion, int *aiCode)
```


---

# Annexes : constantes et index

## Constantes sélectionnées

### Alertes

```cpp
ALERT_CONNECTION_BROKEN = 3;
ALERT_CONNECTION_CLOSED = 2;
ALERT_CONNECTION_OPENED = 1;
ALERT_FORCED_LOGOUT = 7;
ALERT_LOGIN_COMPLETE = 4;
ALERT_LOGIN_FAILED = 5;
ALERT_QUIET_HEARTBEAT = 8;
ALERT_SERVICE_ERROR = 6;
ALERT_SHUTDOWN_SIGNAL = 11;
ALERT_TRADING_DISABLED = 9;
ALERT_TRADING_ENABLED = 10;
```

### Connexions

```cpp
INTRADAY_HISTORY_CONNECTION_ID = 4;
MARKET_DATA_CONNECTION_ID = 1;
PNL_CONNECTION_ID = 3;
REPOSITORY_CONNECTION_ID = 5;
TRADING_SYSTEM_CONNECTION_ID = 2;
```

### Flags_market_data

```cpp
MD_ALL = MD_PRINTS | MD_QUOTES | MD_BEST | MD_CLOSE | MD_PRINTS_COND | MD_SETTLEMENT | MD_OPEN | MD_MARKET_MODE | MD_HIGH_LOW | MD_TRADE_VOLUME | MD_CLOSING_INDICATOR | MD_OPENING_INDICATOR | MD_OPEN_INTEREST | MD_REF_DATA | MD_MID_PRICE | MD_HIGH_BID_LOW_ASK | MD_PRICE_LIMIT | MD_PROJECTED_SETTLEMENT;
MD_BEST = 0x04;
MD_CLOSE = 0x08;
MD_CLOSING_INDICATOR = 0x400;
MD_HIGH_BID_LOW_ASK = 0x8000;
MD_HIGH_LOW = 0x100;
MD_HISTORY_CB = 3;
MD_IMAGE_CB = 1;
MD_MARKET_MODE = 0x80;
MD_MID_PRICE = 0x4000;
MD_NONE = 0x00;
MD_OPEN = 0x40;
MD_OPENING_INDICATOR = 0x800;
MD_OPEN_INTEREST = 0x1000;
MD_PRICE_LIMIT = 0x10000;
MD_PRINTS = 0x01;
MD_PRINTS_COND = 0x10;
MD_PROJECTED_SETTLEMENT = 0x20000;
MD_QUOTES = 0x02;
MD_REF_DATA = 0x2000;
MD_SETTLEMENT = 0x20;
MD_TRADE_VOLUME = 0x200;
MD_UPDATE_CB = 2;
```

### Types_barres

```cpp
BAR_TYPE_DAILY = 5;
BAR_TYPE_MINUTE = 4;
BAR_TYPE_NONE = 0;
BAR_TYPE_RANGE = 1;
BAR_TYPE_SECOND = 7;
BAR_TYPE_TICK = 2;
BAR_TYPE_VOLUME = 3;
BAR_TYPE_WEEKLY = 6;
```

### Opérateurs_comparaison

```cpp
OP_EQUAL_TO = (1);
OP_GREATER_THAN = (3);
OP_GREATER_THAN_OR_EQUAL_TO = (5);
OP_LESS_THAN = (2);
OP_LESS_THAN_OR_EQUAL_TO = (4);
OP_NONE = (0);
OP_NOT_EQUAL_TO = (6);
OP_NOT_GREATER_THAN = (7);
OP_NOT_LESS_THAN = (8);
```

## Index synthétique des classes

### Administration, recherche et rejeu

- `AgreementInfo` — `classRApi_1_1AgreementInfo.html`
- `AgreementListInfo` — `classRApi_1_1AgreementListInfo.html`
- `AlertInfo` — `classRApi_1_1AlertInfo.html`
- `EnvironmentInfo` — `classRApi_1_1EnvironmentInfo.html`
- `EnvironmentListInfo` — `classRApi_1_1EnvironmentListInfo.html`
- `EnvironmentVariable` — `classRApi_1_1EnvironmentVariable.html`
- `RApi::SearchTerm` — `classRApi_1_1SearchTerm.html`
- `SearchTerm::bCaseSensitive` — `classRApi_1_1SearchTerm.html`
- `SearchTerm::sTerm` — `classRApi_1_1SearchTerm.html`

### Autres structures

- `AggregatorInfo` — `classRApi_1_1AggregatorInfo.html`
- `AutoLiquidateInfo` — `classRApi_1_1AutoLiquidateInfo.html`
- `BinaryContractListInfo` — `classRApi_1_1BinaryContractListInfo.html`
- `ClosingIndicatorInfo` — `classRApi_1_1ClosingIndicatorInfo.html`
- `EasyToBorrowInfo` — `classRApi_1_1EasyToBorrowInfo.html`
- `EasyToBorrowListInfo` — `classRApi_1_1EasyToBorrowListInfo.html`
- `IbListInfo` — `classRApi_1_1IbListInfo.html`
- `RCallbacks::ClosingIndicator()` — `classRApi_1_1RCallbacks.html`
- `REngine::login()` — `classRApi_1_1REngine.html`
- `REngine::logout` — `classRApi_1_1REngine.html`
- `REngine::logout()` — `classRApi_1_1REngine.html`
- `REngine::sendOcaList` — `classRApi_1_1REngine.html`
- `REngine::subscribe()` — `classRApi_1_1REngine.html`
- `REngine::subscribeByUnderlying()` — `classRApi_1_1REngine.html`

### Données de marché et référentiel

- `AskInfo` — `classRApi_1_1AskInfo.html`
- `AuxRefDataInfo` — `classRApi_1_1AuxRefDataInfo.html`
- `BarInfo` — `classRApi_1_1BarInfo.html`
- `BarInfo::dSpecifiedRange` — `classRApi_1_1BarInfo.html`
- `BarInfo::iSpecifiedMinutes` — `classRApi_1_1BarInfo.html`
- `BarInfo::iSpecifiedTicks` — `classRApi_1_1BarInfo.html`
- `BarInfo::sSpecifiedDate` — `classRApi_1_1BarInfo.html`
- `BarParams` — `classRApi_1_1BarParams.html`
- `BarReplayInfo` — `classRApi_1_1BarReplayInfo.html`
- `BidInfo` — `classRApi_1_1BidInfo.html`
- `CloseMidPriceInfo` — `classRApi_1_1CloseMidPriceInfo.html`
- `ClosePriceInfo` — `classRApi_1_1ClosePriceInfo.html`
- `DboBookRebuildInfo` — `classRApi_1_1DboBookRebuildInfo.html`
- `DboInfo` — `classRApi_1_1DboInfo.html`
- `EndQuoteInfo` — `classRApi_1_1EndQuoteInfo.html`
- `EquityOptionStrategyListInfo` — `classRApi_1_1EquityOptionStrategyListInfo.html`
- `ExchangeListInfo` — `classRApi_1_1ExchangeListInfo.html`
- `HighBidPriceInfo` — `classRApi_1_1HighBidPriceInfo.html`
- `HighPriceInfo` — `classRApi_1_1HighPriceInfo.html`
- `HighPriceLimitInfo` — `classRApi_1_1HighPriceLimitInfo.html`
- `InstrumentByUnderlyingInfo` — `classRApi_1_1InstrumentByUnderlyingInfo.html`
- `InstrumentSearchInfo` — `classRApi_1_1InstrumentSearchInfo.html`
- `LimitOrderBookInfo` — `classRApi_1_1LimitOrderBookInfo.html`
- `LowAskPriceInfo` — `classRApi_1_1LowAskPriceInfo.html`
- `LowPriceInfo` — `classRApi_1_1LowPriceInfo.html`
- `LowPriceLimitInfo` — `classRApi_1_1LowPriceLimitInfo.html`
- `MarketModeInfo` — `classRApi_1_1MarketModeInfo.html`
- `MidPriceInfo` — `classRApi_1_1MidPriceInfo.html`
- `OpenInterestInfo` — `classRApi_1_1OpenInterestInfo.html`
- `OpenPriceInfo` — `classRApi_1_1OpenPriceInfo.html`
- `OpeningIndicatorInfo` — `classRApi_1_1OpeningIndicatorInfo.html`
- `OptionListInfo` — `classRApi_1_1OptionListInfo.html`
- `OrderTradeCorrectReport` — `classRApi_1_1OrderTradeCorrectReport.html`
- `PnlInfo::dAvgOpenFillPrice` — `classRApi_1_1PnlInfo.html`
- `PnlInfo::eAvgOpenFillPrice` — `classRApi_1_1PnlInfo.html`
- `PriceIncrInfo` — `classRApi_1_1PriceIncrInfo.html`
- `PriceIncrRow` — `classRApi_1_1PriceIncrRow.html`
- `ProductRmsInfo` — `classRApi_1_1ProductRmsInfo.html`
- `ProductRmsListInfo` — `classRApi_1_1ProductRmsListInfo.html`
- `ProjectedSettlementPriceInfo` — `classRApi_1_1ProjectedSettlementPriceInfo.html`
- `QuoteCancelParams` — `classRApi_1_1QuoteCancelParams.html`
- `QuoteInfo` — `classRApi_1_1QuoteInfo.html`
- `QuoteParams` — `classRApi_1_1QuoteParams.html`
- `QuoteReplayInfo` — `classRApi_1_1QuoteReplayInfo.html`
- `QuoteReport` — `classRApi_1_1QuoteReport.html`
- `RApi::InstrumentSearchInfo` — `classRApi_1_1InstrumentSearchInfo.html`
- `RCallbacks::AskQuote()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::AuxRefData()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::BestAskQuote()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::BestBidAskQuote()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::BestBidQuote()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::BidQuote()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::CloseMidPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::ClosePrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::EndQuote()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::HighBidPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::HighPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::HighPriceLimit()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::LimitOrderBook()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::LowAskPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::LowPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::LowPriceLimit()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::MarketMode()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::MidPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::OpenInterest()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::OpenPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::OpeningIndicator()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::ProjectedSettlementPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::RefData()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::SettlementPrice()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::TradeCondition()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::TradePrint()` — `classRApi_1_1RCallbacks.html`
- `RCallbacks::TradeVolume()` — `classRApi_1_1RCallbacks.html`
- `REngine::replayBars()` — `classRApi_1_1REngine.html`
- `REngine::searchInstrument()` — `classRApi_1_1REngine.html`
- `REngine::subscribeBar()` — `classRApi_1_1REngine.html`
- `RefDataInfo` — `classRApi_1_1RefDataInfo.html`
- `ReplayBarParams` — `classRApi_1_1ReplayBarParams.html`
- `SettlementPriceInfo` — `classRApi_1_1SettlementPriceInfo.html`
- `StrategyInfo` — `classRApi_1_1StrategyInfo.html`
- `StrategyLegInfo` — `classRApi_1_1StrategyLegInfo.html`
- `StrategyListInfo` — `classRApi_1_1StrategyListInfo.html`
- `TradeInfo` — `classRApi_1_1TradeInfo.html`
- `TradeReplayInfo` — `classRApi_1_1TradeReplayInfo.html`
- `TradeRouteInfo` — `classRApi_1_1TradeRouteInfo.html`
- `TradeRouteListInfo` — `classRApi_1_1TradeRouteListInfo.html`
- `TradeVolumeInfo` — `classRApi_1_1TradeVolumeInfo.html`
- `VolumeAtPriceInfo` — `classRApi_1_1VolumeAtPriceInfo.html`

### Noyau

- `AdmCallbacks` — `classRApi_1_1AdmCallbacks.html`
- `LoginParams` — `classRApi_1_1LoginParams.html`
- `OmneException` — `classOmneException.html`
- `OmneTimerCB` — `classOmneEngineSpace_1_1OmneTimerCB.html`
- `RCallbacks` — `classRApi_1_1RCallbacks.html`
- `REngine` — `classRApi_1_1REngine.html`
- `REngineParams` — `classRApi_1_1REngineParams.html`
- `SearchTerm` — `classRApi_1_1SearchTerm.html`

### Trading, comptes et état

- `AccountInfo` — `classRApi_1_1AccountInfo.html`
- `AccountListInfo` — `classRApi_1_1AccountListInfo.html`
- `AccountUpdateInfo` — `classRApi_1_1AccountUpdateInfo.html`
- `AccountUpdateInfo::sAction` — `classRApi_1_1AccountUpdateInfo.html`
- `AssignedUserListInfo` — `classRApi_1_1AssignedUserListInfo.html`
- `BracketInfo` — `classRApi_1_1BracketInfo.html`
- `BracketParams` — `classRApi_1_1BracketParams.html`
- `BracketParams::sOrderOperationType` — `classRApi_1_1BracketParams.html`
- `BracketReplayInfo` — `classRApi_1_1BracketReplayInfo.html`
- `BracketTier` — `classRApi_1_1BracketTier.html`
- `BracketTierModifyInfo` — `classRApi_1_1BracketTierModifyInfo.html`
- `ExecutionReplayInfo` — `classRApi_1_1ExecutionReplayInfo.html`
- `LimitOrderParams` — `classRApi_1_1LimitOrderParams.html`
- `LineInfo` — `classRApi_1_1LineInfo.html`
- `MarketOrderParams` — `classRApi_1_1MarketOrderParams.html`
- `ModifyLimitOrderParams` — `classRApi_1_1ModifyLimitOrderParams.html`
- `ModifyOrderParams` — `classRApi_1_1ModifyOrderParams.html`
- `ModifyStopLimitOrderParams` — `classRApi_1_1ModifyStopLimitOrderParams.html`
- `ModifyStopMarketOrderParams` — `classRApi_1_1ModifyStopMarketOrderParams.html`
- `OrderBustReport` — `classRApi_1_1OrderBustReport.html`
- `OrderCancelReport` — `classRApi_1_1OrderCancelReport.html`
- `OrderFailureReport` — `classRApi_1_1OrderFailureReport.html`
- `OrderFillReport` — `classRApi_1_1OrderFillReport.html`
- `OrderHistoryDatesInfo` — `classRApi_1_1OrderHistoryDatesInfo.html`
- `OrderModifyReport` — `classRApi_1_1OrderModifyReport.html`
- `OrderNotCancelledReport` — `classRApi_1_1OrderNotCancelledReport.html`
- `OrderNotModifiedReport` — `classRApi_1_1OrderNotModifiedReport.html`
- `OrderParams` — `classRApi_1_1OrderParams.html`
- `OrderRejectReport` — `classRApi_1_1OrderRejectReport.html`
- `OrderReplayInfo` — `classRApi_1_1OrderReplayInfo.html`
- `OrderReport` — `classRApi_1_1OrderReport.html`
- `OrderStatusReport` — `classRApi_1_1OrderStatusReport.html`
- `OrderTriggerPulledReport` — `classRApi_1_1OrderTriggerPulledReport.html`
- `OrderTriggerReport` — `classRApi_1_1OrderTriggerReport.html`
- `PasswordChangeInfo` — `classRApi_1_1PasswordChangeInfo.html`
- `PnlInfo` — `classRApi_1_1PnlInfo.html`
- `PnlReplayInfo` — `classRApi_1_1PnlReplayInfo.html`
- `PositionExitInfo` — `classRApi_1_1PositionExitInfo.html`
- `RApi::LineInfo::sReleaseCommand` — `classRApi_1_1LineInfo.html`
- `RApi::OrderParams::sReleaseCommand` — `classRApi_1_1OrderParams.html`
- `RCallbacks::PnlUpdate` — `classRApi_1_1RCallbacks.html`
- `REngine::getAccounts()` — `classRApi_1_1REngine.html`
- `RmsInfo` — `classRApi_1_1RmsInfo.html`
- `RmsInfo::sStatus` — `classRApi_1_1RmsInfo.html`
- `SingleOrderReplayInfo` — `classRApi_1_1SingleOrderReplayInfo.html`
- `SodReport` — `classRApi_1_1SodReport.html`
- `StopLimitOrderParams` — `classRApi_1_1StopLimitOrderParams.html`
- `StopMarketOrderParams` — `classRApi_1_1StopMarketOrderParams.html`
- `UserDefinedSpreadCreateInfo` — `classRApi_1_1UserDefinedSpreadCreateInfo.html`
- `UserInfo` — `classRApi_1_1UserInfo.html`
- `UserInfo::sAction` — `classRApi_1_1UserInfo.html`
- `UserInfo::sStatus` — `classRApi_1_1UserInfo.html`
- `UserListInfo` — `classRApi_1_1UserListInfo.html`
- `UserProfileInfo` — `classRApi_1_1UserProfileInfo.html`


