# TradingView_V0.1.0
# Trading algorithm***

# https://static.tradingview.com/static/bundles/broker-rest-api.45f1b90af960162ffc90.yaml
openapi:
  3.0.0
info:
  version: 1.4.6
  title: TradingView REST API Specification for Brokers
  description: |
    ## Overview
      This API is to be implemented by the Brokers in order to connect their backend systems to TradingView, that acts as a frontend.

      Check the [info page](https://www.tradingview.com/brokerage-integration/) for more info and use the contact form there if you have any questions.

      ### Types of requests
      There are two types of requests â€” client and server.
      Client requests are executed at the browser. Server requests are initiated from the TradingView servers.
      If your integration does not imply brokerage data stream connection to the TradingView website -
      then there won't be any server requests.

      #### Clients requests
      The two possible ways of sending updates to the client's side are either done by making use of client regularly polling endpoints,
      or by opening a streaming connection and pushing updates.

      ##### HTTP requests
      From the browser TradingView requests the info (list of orders and positions, balance info, etc.) from the brokerâ€™s server.
      The requests are sent periodically and the intervals can be set by using the [/config](#operation/getConfiguration) endpoint.
      After that, TradingView compares the new data with the previous answer and calculates the difference.
      If the status of the order/position changes or new data appears - the user will see a notification and the changes
      will display in the Account manager on the website.

      Requests to the [endpoints](/rest-api-spec/#tag/Trading) for placing/modifying orders, positions closing, etc. occur only after actions made by the user.

      The [/quotes](#operation/getQuotes) endpoint retrieves the current bid/ask from the broker.
      The [/depth](#operation/getDepth) endpoint retrieves Level 2 market data.

      ##### HTTP streaming
      If the integration uses HTTP streaming endpoints, the connections to them are established once at the start of a session.
      The http-streaming consists of messages separated with line breaks.
      The content of the messages must be in Compact JSON format, i.e. should not contain any line breaks, except a line break at the very end
      which will act as a message separator.

      There are four types of messages:
      - Entities snapshot
      - Entities updates
      - Ping
      - Error

      Please refer to the streaming endpoints for more details.

      #### Server requests
      In case if a Broker provides any Forex or CFD trading access for its clients it will require connection
      of its own market data at TradingView. In order to make it possible, you will need to implement the following endpoints -
        [/symbol_info](#operation/getSymbolInfo), [/history](#operation/getHistory) and [/streaming](#operation/streaming).
      Data requests are sent from different TradingView servers. Usually, at least 4 servers are used.
      The historical data is cached on TradingView servers and loaded to the client browser from our servers.

      ### Restricting access to data
      By default, the broker symbols will be available in the symbol search at TradingView and all the community
      will have access to your data streams without any limitation. In order to limit the access to your data streams
      please use the following endpoints - [/groups](#operation/getGroups) and [/permissions](#operation/getPermissions).

      You can find more information about restricting access to the data in the description of these endpoints.

      ### Change log
      1.4.6 Added 'modify_individual_position' and 'close_individual_position' transaction types to the [/trackLatency](#operation/trackLatency) endpoint.

      1.4.5 Added support for `refresh_token_expires_in` parameter for OAuth 2 Code Flow.

      1.4.4 Added support for individual and net position streaming, added two new endpoints:
        [/stream/individualPositions](#operation/streamIndividualPositions) a streaming alternative to the [/individualPositions](#operation/getIndividualPositions) endpoint.
        [/stream/netPositions](#operation/streamNetPositions) a streaming alternative to the [/netPositions](#operation/getIndividualPositions) endpoint.

      1.4.3 Added `marginAvailable` field to the `AccountState` parameters.

      1.4.2 Added `amount` field to [/individualPositions](#operation/closeIndividualPosition) endpoint. Added new `supportPartialCloseIndividualPosition` flag.

      1.4.1 Added two new endpoints:
        [/stream/state](#operation/streamState). A streaming alternative to the [/state](#operation/getState) endpoint.
        [/stream/quotes](#operation/streamQuotes). A streaming alternative to the [/quotes](#operation/getQuotes) endpoint.

      1.4.0 Added support for http-streaming and two new endpoints: [/stream/orders](#operation/streamOrders) and [/stream/positions](#operation/streamPositions).

      1.3.35. Added support for net and individual positions. Added the following endpoints:
        [/netPositions](#operation/getNetPositions), [/individualPositions](#operation/getIndividualPositions),
        [/closeNetPosition](#operation/closeNetPosition), [/closeIndividualPosition](#operation/closeIndividualPosition)
        and [/modifyIndividualPosition](#operation/modifyIndividualPosition).
        Added new flag `supportIndividualPositionBrackets`.

      1.3.34. Added track latency support. Added new `supportTrackLatency` flag. Added new endpoint:
        [/trackLatency](#operation/trackLatency).

      1.3.33. Added new flag `supportStopLimitOrdersInBothDirections`.

      1.3.32. Added new flag `supportLeverageButton`.

      1.3.31. The locale parameter for the /authorization endpoint is now required only for trading integration.

      1.3.30. Added new `currentAsk` and `currentBid` fields to the [/modifyPosition](#operation/modifyPosition) endpoint.

      1.3.29. Added custom checkbox support.
        Added new `checkbox` field to the [/accounts](#operation/getAccounts) and [/instruments](#operation/getInstruments) endpoints
        for the `orderDialogCustomFields` object.

      1.3.28. Added new `variableMinTick` field to the [/instruments](#operation/getInstruments) endpoint.

      1.3.27. Added `hardToBorrow`, `notShortable`, `halted` parameters to the [/quotes](#operation/getQuotes) endpoint.

      1.3.26. In [/symbol_info](#operation/getSymbolInfo), the `minmov2` property renamed to `minmovement2`, the `Etc/UTC`
        timezone added. In [/history ](#operation/getHistory), the `HistoryNextBarResponse` response changed to
        `HistoryEmptyBarResponse`, the `countback` query parameter became deprecated.
        In [/accounts](#operation/getAccounts), added the `isVerified` property.

      1.3.25. Added `locale` parameter to the [/authorize](#operation/authorize) endpoint.

      1.3.24. Added leverage support. Added new `supportLeverage` flag. Added three new endpoints:
        [/getLeverage](#operation/getLeverage), [/setLeverage](#operation/setLeverage) and [/previewLeverage](#operation/previewLeverage).

      1.3.23. Changed description of `forceUserEnterInitialValue` flag in the `OrderDialogCustomFields` parameters.

      1.3.22. Added new flags - `supportModifyOrderPrice` and `supportModifyBrackets`. The `supportModifyOrder` flag became deprecated.

      1.3.21. Added new `units` field to the [/instruments](#operation/getInstruments) endpoint.

      1.3.20. Added new `supportTrailingStop` flag. Added support for trailing stops for orders and positions.

      1.3.19. Added new `logout` endpoint, `supportLogout` flag.

      1.3.18. Added new `supportPartialClosePosition` flag and `amount` field to the [/closePosition](#operation/closePosition)
        endpoint parameters.

      1.3.17. Added `stopPercent` and `limitPercent` validation rules to the [/instruments](#operation/getInstruments) endpoint.

      1.3.16. Added new `supportOrderHistoryCustomFields` flag and `orderHistoryCustomFields` field to the `ui` object
        in the [/accounts](#operation/getAccounts) endpoint and [/config](#operation/getConfiguration).

      1.3.15. Added `supportedOrderTypes` field to the `Duration` parameters.

      1.3.14. Added `isCapitalize` field to the `positionCustomFields` and `orderCustomFields` parameters.

      1.3.13. Added rules parameter to the [/instruments](#operation/getInstruments) endpoint.

      1.3.12. Added `prefix` field to the `Account` parameters.

      1.3.11. Changed description for [/placeOrder](#operation/placeOrder), [/modifyOrder](#operation/modifyOrder),
        [/previewOrder](#operation/previewOrder), `OrderDialogCustomFields` and `customFields` field in `OrderCommon`.

      1.3.10. Added `locale` query parameter to [/getOrders](#operation/getOrders), [/placeOrder](#operation/placeOrder),
        [/modifyOrder](#operation/modifyOrder), [/cancelOrder](#operation/cancelOrder), [/getPositions](#operation/getPositions),
        [/modifyPosition](#operation/modifyPosition), [/closePosition](#operation/closePosition),
        [/getExecutions](#operation/getExecutions), [/getOrdersHistory](#operation/getOrdersHistory),
        [/getQuotes](#operation/getQuotes), [/getDepth](#operation/getDepth) and [/getBalances](#operation/getBalances) endpoints.

      1.3.9. Added `orderId`, `isClose`, `positionId` and `commission` fields to Execution.

      1.3.8. Added `id` field to the [/previewOrder](#operation/previewOrder) endpoint parameters.

      1.3.7. Added `mutable` field to the `OrderDialogCustomFields` parameters.

      1.3.6. Added `lang` query parameter to OAuth authorization request.

      1.3.5. Added `accountId` query parameter to `depth` endpoint.

      1.3.4. Added Order dialog customization opportunity on the instrument basis. Moved `OrderDialogCustomFields` to
        the `ui` object in the [/accounts](#operation/getAccounts) endpoint.

      1.3.3. Added new `supportStopOrdersInBothDirections` flag.

      1.3.2. Added new `previewOrder` endpoint, `supportPlaceOrderPreview` and `supportModifyOrderPreview ` flags.

      1.3.1. Added OAuth 2 Code Flow.

      1.3.0. Added overriding Account manager and Durations configuration on Account basis.

      1.2.5. Added `reserved`, `value` and `valueCurrency` fields to Crypto Balances.

      1.2.4. Added default values to the account flags. Added `supportMarketBrackets` account flag.

      1.2.3. Added current ask/bid fields to the parameters of the order [placement](#operation/placeOrder)
      and [modification](#operation/modifyPosition) requests.

      1.2.2. Added supportPartialOrderExecution flag.

      1.2.1. Added support for position's and order's Custom fields. Removed `fixedWidth` and `sortable` fields from `AccountManagerColumn`.

      1.2.0. Introducing new Quote response. Deprecation of streaming Bid and Ask responses.
      All new integrations should use the Quote response to provide ask/bid values.
      Added supportMarketOrders, supportLimitOrders, supportStop orders account flags.
      Added informational message to order and position.

      1.1.3. Added support for reverse of the position.

      1.1.2. Added support for custom Account Summary Row.

      1.1.1. Added `type` field to [/accounts](#operation/getAccounts) endpoint.

      1.1.0. Refactor, added examples.

servers:
  - url: https://your-rest-implementation.com/api
    description: Your REST API implementation url. Change it to your real REST API url.

paths:

  /authorize:
    post:
      summary: Authorize
      operationId: authorize
      tags:
        - Authorization
      description: Username and password authentication.
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              type: object
              properties:
                login:
                  type: string
                  description: User login.
                password:
                  type: string
                  description: User password.
                locale:
                  $ref: '#/components/schemas/Locale'
              required:
                - login
                - password
            example: {
              "login": "user1",
              "password": "dfkjhoijogpoi",
              "locale": en,
            }
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/AuthorizeResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /logout:
    post:
      summary: Logout
      operationId: logout
      tags:
        - Authorization
      description: Send logout if the supportLogout flag is set as true.
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /config:
    get:
      summary: Configuration
      operationId: getConfiguration
      tags:
        - Broker Configuration
      description: Get localized configuration.
      parameters:
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/ConfigResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /mapping:
    get:
      summary: Mapping
      operationId: getMapping
      tags:
        - Broker Configuration
      description: |
        Return all broker instruments with corresponding TradingView instruments.
        It is required to add a Broker to TradingView.com.
        Please note that this endpoint works without authorization!
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/MappingResponse'

  /accounts:
    get:
      summary: Accounts
      operationId: getAccounts
      tags:
        - Account
      description: Get a list of accounts owned by the user.
      parameters:
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/AccountResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/instruments:
    get:
      summary: Instruments
      operationId: getInstruments
      tags:
        - Account
      description: |
        Get the list of the instruments that are available for trading with the specified account.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/InstrumentsResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/state:
    get:
      summary: State
      operationId: getState
      tags:
        - Account
      description: Get account information.
        Consider using [stream/state](#operation/streamState) instead, as it can decrease the workload on the servers.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/AccountStateResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/stream/state:
    get:
      summary: Stream State
      operationId: streamState
      tags:
        - Account
      description: Stream account information.
        Unlike [/stream/orders](#operation/streamOrders) or [/stream/positions](#operation/streamPositions)
        the whole state object must be sent to the client every time an update occurs.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/AccountStateMessage'
                  - $ref: '#/components/schemas/PingMessage'
                  - $ref: '#/components/schemas/ErrorMessage'

  /accounts/{accountId}/orders:
    get:
      summary: Orders
      operationId: getOrders
      tags:
        - Account
      description: |
        Get current session orders for the account. It also includes working orders from previous sessions.
        Filled/cancelled/rejected orders should be included in the list till the end of the session.
        Consider using [stream/orders](#operation/streamOrder) instead, as it can decrease the workload on the servers.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/OrdersResponse'
                  - $ref: '#/components/schemas/ErrorResponse'
    post:
      summary: Place Order
      operationId: placeOrder
      tags:
        - Trading
      description: |
        Place a new order. Custom `Order dialog` fields defined in the [/accounts](#operation/getAccounts) are sent along with the standard fields in the order object.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
        - $ref: '#/components/parameters/requestId'
        - $ref: '#/components/parameters/confirmId'
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              allOf:
                - $ref: '#/components/schemas/PlacePreviewOrderCommon'
                - $ref: '#/components/schemas/PlaceModifyPreviewOrderCommon'
              required:
                - instrument
                - qty
                - side
                - type
                - currentAsk
                - currentBid
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/PostOrderResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/stream/orders:
    get:
      summary: Stream Orders
      operationId: streamOrders
      tags:
        - Account
      description:
        Stream orders of the current session for the account. It also includes working orders from the previous sessions.
        A snapshot of all orders should be sent as the first message in the stream.
        After that the server must only send updates. It's not required to send the list of all orders in every message.
        If there are updated fields in an order, the server must send the whole order, including unchanged fields.
        If there were no updates for 5 seconds, a ping message must be sent.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/OrdersMessage'
                  - $ref: '#/components/schemas/PingMessage'
                  - $ref: '#/components/schemas/ErrorMessage'

  /accounts/{accountId}/positions:
    get:
      summary: Positions
      operationId: getPositions
      tags:
        - Account
      description: Get positions for an account.
        Consider using [stream/positions](#operation/streamPositions) instead, as it can decrease the workload on the servers.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/PositionsResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/stream/positions:
    get:
      summary: Stream Positions
      operationId: streamPositions
      tags:
        - Account
      description:
        Stream positions for an account.
        A snapshot of all positions should be sent as the first message in the stream.
        After that the server must only send updates. It's not required to send the list of all positions in every message.
        If there are updated fields in a position, the server must send the whole position, including unchanged fields.
        If there were no updates for 5 seconds, a ping message must be sent.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/PositionsMessage'
                  - $ref: '#/components/schemas/PingMessage'
                  - $ref: '#/components/schemas/ErrorMessage'

  /accounts/{accountId}/netPositions:
    get:
      summary: Net Positions
      operationId: getNetPositions
      tags:
        - Account
      description: Get net positions for an account.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/NetPositionsResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/stream/netPositions:
    get:
      summary: Stream Net Positions
      operationId: streamNetPositions
      tags:
        - Account
      description:
        Stream net positions for an account.
        A snapshot of all net positions should be sent as the first message in the stream.
        After that the server must only send updates. It's not required to send the list of all net positions in every message.
        If there are updated fields in a net position, the server must send the whole net position, including unchanged fields.
        If there were no updates for 5 seconds, a ping message must be sent.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/NetPositionMessage'
                  - $ref: '#/components/schemas/PingMessage'
                  - $ref: '#/components/schemas/ErrorMessage'

  /accounts/{accountId}/individualPositions:
    get:
      summary: Individual Positions
      operationId: getIndividualPositions
      tags:
        - Account
      description: Get individual positions for an account
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/IndividualPositionsResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/stream/individualPositions:
    get:
      summary: Stream Individual Positions
      operationId: streamIndividualPositions
      tags:
        - Account
      description:
        Stream individual positions for an account.
        A snapshot of all individual positions should be sent as the first message in the stream.
        After that the server must only send updates. It's not required to send the list of all individual positions in every message.
        If there are updated fields in a individual position, the server must send the whole individual position, including unchanged fields.
        If there were no updates for 5 seconds, a ping message must be sent.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/IndividualPositionMessage'
                  - $ref: '#/components/schemas/PingMessage'
                  - $ref: '#/components/schemas/ErrorMessage'

  /accounts/{accountId}/balances:
    get:
      summary: Balances
      operationId: getBalances
      tags:
        - Account
      description: |
        Get crypto balances for an account.
        Balances are displayed as the first table of the Account Summary tab.
        Used for crypto currencies only.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/CryptoBalancesResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/executions:
    get:
      summary: Executions
      operationId: getExecutions
      tags:
        - Account
      description: |
        Get a list of executions (i.e. fills or trades) for an account and an
        instrument. Executions are displayed on a chart.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
        - name: instrument
          in: query
          description: Broker instrument name.
          schema:
            type: string
          required: true
        - name: maxCount
          description: Maximum count of executions to return.
          in: query
          schema:
            type: number
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/ExecutionsResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/ordersHistory:
    get:
      summary: Orders History
      operationId: getOrdersHistory
      tags:
        - Account
      description: |
        Get order history for an account. It is expected that returned orders will have a final status (`rejected`,
        `filled`, `cancelled`). This endpoint is optional. If you don't support orders history, please set
        the `supportOrdersHistory` flag to `false`.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
        - name: maxCount
          description: Maximum count of orders to return.
          in: query
          schema:
            type: number
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/OrdersHistoryResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/getLeverage:
    post:
      summary: Get Leverage
      operationId: getLeverage
      tags:
        - Account
      description: |
        Request to this endpoint will be sent when the user opens the order ticket or changes any of the symbol, order type, side and any of the custom fields in the order ticket.
        Custom `Order dialog` fields defined in the [/accounts](#operation/getAccounts) are sent along with the standard fields in the order object.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        content:
          application/x-www-form-urlencoded:
            schema:
              type: object
              properties:
                instrument:
                  description: Broker instrument name.
                  type: string
                  example: BTCPERP
                side:
                  description: Current order side in the order ticket.
                  type: string
                  enum:
                    - buy
                    - sell
                orderType:
                  description: Current order type selected in the order ticket.
                  type: string
                  enum:
                    - market
                    - stop
                    - limit
                    - stoplimit
              required:
                - instrument
                - side
                - orderType
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/GetLeverageResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/setLeverage:
    post:
      summary: Set Leverage
      operationId: setLeverage
      tags:
        - Account
      description: |
        Will be sent when the user confirms changing the leverage.
        Additional "leverage" field will be added to the payload, the value of which was set by the user.
        Custom `Order dialog` fields defined in the [/accounts](#operation/getAccounts) are sent along with the standard fields in the order object.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              type: object
              properties:
                instrument:
                  description: Broker instrument name.
                  type: string
                  example: BTCPERP
                side:
                  description: Current order side in the order ticket.
                  type: string
                  enum:
                    - buy
                    - sell
                orderType:
                  description: Current order type selected in the order ticket.
                  type: string
                  enum:
                    - market
                    - stop
                    - limit
                    - stoplimit
                leverage:
                  description: Leverage value set by the user
                  type: number
                  example: 25
              required:
                - instrument
                - side
                - orderType
                - leverage
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SetLeverageResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/previewLeverage:
    post:
      summary: Preview Leverage
      operationId: previewLeverage
      tags:
        - Account
      description: |
        Will be sent when the user is editing the leverage.
        Custom `Order dialog` fields defined in the [/accounts](#operation/getAccounts) are sent along with the standard fields in the order object.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        content:
          application/x-www-form-urlencoded:
            schema:
              type: object
              properties:
                instrument:
                  description: Broker instrument name.
                  type: string
                  example: BTCPERP
                side:
                  description: Current order side in the order ticket.
                  type: string
                  enum:
                    - buy
                    - sell
                orderType:
                  description: Current order type selected in the order ticket.
                  type: string
                  enum:
                    - market
                    - stop
                    - limit
                    - stoplimit
                leverage:
                  description: Leverage value set by user
                  type: number
                  example: 25
              required:
                - instrument
                - side
                - orderType
                - leverage
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/PreviewLeverageResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/orders/{orderId}:
    put:
      summary: Modify Order
      operationId: modifyOrder
      tags:
        - Trading
      description: Modify an existing order. Custom `Order dialog` fields defined in the [/accounts](#operation/getAccounts) are sent along with the standard fields in the order object.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/orderId'
        - $ref: '#/components/parameters/locale'
        - $ref: '#/components/parameters/confirmId'
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              allOf:
                - $ref: '#/components/schemas/PlaceModifyPreviewOrderCommon'
              required:
                - instrument
                - qty
                - side
                - type
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/SuccessResponseWithTransactionId'
                  - $ref: '#/components/schemas/ErrorResponse'
    delete:
      summary: Cancel Order
      operationId: cancelOrder
      tags:
        - Trading
      description: Cancel an existing order.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/orderId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/SuccessResponseWithTransactionId'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/previewOrder:
    post:
      summary: Preview Order
      operationId: previewOrder
      tags:
        - Trading
      description: |
        Get estimated cost, commission and other information for an order without the order actually being placed or modified. This information is displayed in the Order Ticket Preview.
        This endpoint is used if supportPlaceOrderPreview and/or supportModifyOrderPreview flag is `true`.
        TradingView displays the following information by itself&#58; symbol, bid/ask, order type, side, quantity, price (except market orders), stop loss, take profit and currency.
        Custom `Order dialog` fields defined in the [/accounts](#operation/getAccounts) are sent along with the standard fields in the order object.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              allOf:
                - $ref: '#/components/schemas/PlaceModifyPreviewOrderCommon'
                - $ref: '#/components/schemas/PlacePreviewOrderCommon'
                - type: object
                  properties:
                    id:
                      type: string
                      description: Identifier of the order that is being modified by the user. This parameter is sent only if `supportModifyOrderPreview` flag is `true`.
                      example: 21698953
              required:
                - instrument
                - qty
                - side
                - type
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/PreviewOrderResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/positions/{positionId}:
    put:
      summary: Modify Position
      operationId: modifyPosition
      tags:
        - Trading
      description: |
        Modify an existing position stop loss or take profit or both.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/positionId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              $ref: '#/components/schemas/ModifyPositionRequestBody'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/SuccessResponseWithTransactionId'
                  - $ref: '#/components/schemas/ErrorResponse'
    delete:
      summary: Close Position
      operationId: closePosition
      tags:
        - Trading
      description: Close an existing position.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/positionId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        content:
          application/x-www-form-urlencoded:
            schema:
              $ref: '#/components/schemas/DeletePositionRequestBody'
            example: {
              "amount": 5,
            }
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/SuccessResponseWithTransactionId'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/netPositions/{netPositionId}:
    delete:
      summary: Close Net Position
      operationId: closeNetPosition
      tags:
        - Trading
      description: Close an existing net position and all of the individual positions on the symbol.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/netPositionId'
        - $ref: '#/components/parameters/locale'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/individualPositions/{individualPositionId}:
    put:
      summary: Modify Individual Position
      operationId: modifyIndividualPosition
      tags:
        - Trading
      description: Modify an existing individual position stop loss or take profit or both.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/individualPositionId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              $ref: '#/components/schemas/ModifyIndividualPositionRequestBody'
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/ErrorResponse'
    delete:
      summary: Close Individual Position
      operationId: closeIndividualPosition
      tags:
        - Trading
      description: Close an individual position.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
        - $ref: '#/components/parameters/individualPositionId'
        - $ref: '#/components/parameters/locale'
      requestBody:
        content:
          application/x-www-form-urlencoded:
            schema:
              $ref: '#/components/schemas/DeleteIndividualPositionRequestBody'
            example: {
              "amount": 5,
            }
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SuccessResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /accounts/{accountId}/trackLatency:
    post:
      summary: Latency Tracking
      operationId: trackLatency
      tags:
        - Trading
      description: |
        Endpoint is used to collect performance and statistical data for a particular request.
        Applies to the following endpoints:
          Place order: [/placeOrder](#operation/placeOrder)
          Modify order: [/modifyOrder](#operation/modifyOrder)
          Cancel order: [/cancelOrder](#operation/cancelOrder)
          Modify position: [/modifyPosition](#operation/modifyPosition)
          Close position: [/closePosition](#operation/closePosition)
        To enable latency tracking functionality the `supportTrackLatency` flag should be set to true in the account config. Please see the [/accounts](#operation/getAccounts) endpoint.
      parameters:
        - $ref: '#/components/parameters/pathAccountId'
      requestBody:
        required: true
        content:
          application/x-www-form-urlencoded:
            schema:
              type: object
              properties:
                transactionId:
                  description: Request identifier for which the performance data is being sent.
                  type: number
                transactionType:
                  description: Type of the transaction
                  type: string
                  enum:
                    - 'place_order'
                    - 'modify_order'
                    - 'cancel_order'
                    - 'modify_position'
                    - 'close_position'
                    - 'modify_individual_position'
                    - 'close_individual_position'
                    - 'close_net_position'
                instrument:
                  description: Instrument.
                  type: string
                qty:
                  description: Placed or modified orders quantity. Not applied to Cancel order / modify position / close position requests.
                  type: number
                side:
                  description: Side.
                  type: string
                  enum:
                    - 'buy'
                    - 'sell'
                ask:
                  description: Current ask price for the instrument that the user sees in the order panel.
                  type: number
                  example: 1.287367
                bid:
                  description: Current bid price for the instrument that the user sees in the order panel.
                  type: number
                  example: 1.28554
                timestamp:
                  description: Timestamp of the price quote user has seen on the UI. In milliseconds.
                  type: number
                  example: 1548406235
                roundTripDuration:
                  description: Request duration in milliseconds.
                  type: number
                  example: 183
              required:
                - transactionId
                - transactionType
                - instrument
                - side
                - ask
                - bid
                - timestamp
                - roundTripDuration
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                  $ref: '#/components/schemas/SuccessResponse'

  /quotes:
    get:
      summary: Quotes
      operationId: getQuotes
      tags:
        - Market Data
      description: |
        Get current prices of the instrument.
        The `bid` and `ask` fields are required, and the `buyPipValue` and `sellPipValue` fields
        are desirable if the account currency is different from the currency of the instrument.
        The same values should be sent for these fields if different values for buying and selling are not supported.
      parameters:
        - $ref: '#/components/parameters/locale'
        - $ref: '#/components/parameters/queryAccountId'
        - name: symbols
          description: Comma separated symbol list.
          in: query
          schema:
            type: string
          required: true
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/QuotesResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /stream/quotes:
    get:
      summary: Stream Quotes
      operationId: streamQuotes
      tags:
        - Market Data
      description: |
        Stream current prices of the instrument.
        The `bid` and `ask` fields are required, and the `buyPipValue` and `sellPipValue` fields
        are desirable if the account currency is different from the currency of the instrument.
        The same values should be sent for these fields if different values for buying and selling are not supported.
      parameters:
        - $ref: '#/components/parameters/locale'
        - $ref: '#/components/parameters/queryAccountId'
        - name: symbols
          description: Comma separated symbol list.
          in: query
          schema:
            type: string
          required: true
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/QuotesMessage'
                  - $ref: '#/components/schemas/PingMessage'
                  - $ref: '#/components/schemas/ErrorMessage'


  /depth:
    get:
      summary: Depth
      operationId: getDepth
      tags:
        - Market Data
      description: Get current depth of market for the instrument. Optional.
      parameters:
        - $ref: '#/components/parameters/locale'
        - $ref: '#/components/parameters/queryAccountId'
        - name: symbol
          description: Instrument name.
          in: query
          schema:
            type: string
          required: true
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/DepthResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /groups:
    get:
      summary: Groups
      operationId: getGroups
      tags:
        - Data Permissions
      description: |
        Get a list of possible groups of symbols.
        A group is a set of symbols that share a common access level. Any user can have access to any number of such groups.
        It is required only if you use groups of symbols in order to restrict access to the instrument's data.

        **IMPORTANT:**
        Please plan your symbol grouping carefully. Groups cannot be deleted, you can only remove all the symbols from there.

        **LIMITATIONS:**
        Each integration is limited to have up to 10 symbol groups.
        Each symbol group is limited to have up to 10K symbols in it.
        You cannot put the same symbol into 2 different groups.

        This endpoint allows you to specify a list of groups, and the [/permissions](#operation/getPermissions) endpoint specifies
        which groups are available for the certain user.

        When TradingView user logs into his broker account - he will gain access to one or more groups,
        depending on the [/permissions](#operation/getPermissions) endpoint.

        At the [/symbol_info](#operation/getSymbolInfo) endpoint TradingView adds the GET argument `group`
        with the name of the group. Thus, TradingView will receive information about which group each symbol belongs to.

      security:
        - PasswordBearer: []
        - ServerOAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/GroupListResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /permissions:
    get:
      summary: Permissions
      operationId: getPermissions
      tags:
        - Data Permissions
      description: |
        Get a list of symbol groups allowed for the user.
        It is only required if you use groups of symbols to restrict access to instrument's data.
      security:
        - PasswordBearer: []
        - OAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/PermissionsResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /symbol_info:
    get:
      summary: Symbol Info
      operationId: getSymbolInfo
      tags:
        - Data Integration
      description: Get a list of all instruments.
      parameters:
        - $ref: '#/components/parameters/group'
      security:
        - PasswordBearer: []
        - ServerOAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/SymbolInfoResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /history:
    get:
      summary: History
      operationId: getHistory
      tags:
        - Data Integration
      description: |
        Request for history bars. Each property of the response object is treated as a table column.

        Data should meet the following requirements:

        - real-time data obtained from the API streaming endpoint must match the historical data, obtained from the
          /history API. The allowed count of mismatched bars (candles) must not exceed 5% for frequently traded symbols,
          otherwise the integration to TradingView is not possible;
        - the data must not include unreasonable price gaps, historical data gaps on 1-minute and Daily-resolutions
          (temporal gaps), obviously incorrect prices (adhesions).

        Bar time for daily bars should be 00:00 UTC and is expected to be a trading day
        (not a day when the session starts).

        Bar time for monthly bars should be 00:00 UTC and is the first trading day of the month.

        If there is no data in the requested time period, you should return an empty response:
        `{"s":"ok","t":[],"o":[],"h":[],"l":[],"c":[],"v":[]}`
      parameters:
        - $ref: '#/components/parameters/symbol'
        - $ref: '#/components/parameters/resolution'
        - $ref: '#/components/parameters/historyFrom'
        - $ref: '#/components/parameters/historyTo'
        - $ref: '#/components/parameters/countback'
      security:
        - PasswordBearer: []
        - ServerOAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/HistorySuccessResponse'
                  - $ref: '#/components/schemas/HistoryEmptyBarResponse'
                  - $ref: '#/components/schemas/ErrorResponse'

  /streaming:
    get:
      summary: Stream of prices
      operationId: streaming
      tags:
        - Data Integration
      description: |
        Stream of prices. Server constantly keeps the connection alive. If the
        connection is broken - the server constantly tries to restore it.
        TradingView establishes up to 4 simultaneous connections to this endpoint and
        expects to get the same data to all of them.
        Transfer mode is `chunked encoding`. The data feed should set `'Transfer-Encoding:
        chunked'` and make sure that all intermediate proxies are set to use this
        mode. All messages are to be ended with `\n`. Data stream should contain
        real-time data only. It shouldn't contain snapshots of data.

        Data feed should provide trades and quotes:
        - If trades are not provided, then data feed should set trades with bid price and bid size (mid price with 0 size in case of Forex).
        - Size is always greater than `0`, except for the correction. In that case size can be `0`.
        - Quote must contain prices of the best ask and the best bid.
        - Daily bars are required if they cannot be built from trades (has-daily should be set to true in the symbol information in that case).

        The broker must remove unnecessary restrictions (firewall, rate limits, etc.) for the set of IP addresses of our servers.

        `StreamingHeartbeatResponse` is a technical message that prevents the feed from unsubscribing from streaming.
        It doesn't affect the price data and should be sent to /streaming every 5 seconds by default. The message
        can be used, for example, when the trading session is closed on weekends or in case of low liquidity
        on the exchange when TradingView does not receive price updates for a long time.

        Please note, that `StreamingAskResponse` and `StreamingBidResponse` are deprecated.
        The `StreamingQuoteResponse` should be used to provide ask / bid data.

      security:
        - PasswordBearer: []
        - ServerOAuth2Bearer: []
      responses:
        200:
          description: response
          content:
            application/json:
              schema:
                oneOf:
                  - $ref: '#/components/schemas/StreamingQuoteResponse'
                  - $ref: '#/components/schemas/StreamingTradeResponse'
                  - $ref: '#/components/schemas/StreamingDailyBarResponse'
                  - $ref: '#/components/schemas/StreamingHeartbeatResponse'
                  - $ref: '#/components/schemas/StreamingAskResponse(deprecated)'
                  - $ref: '#/components/schemas/StreamingBidResponse(deprecated)'

components:
  securitySchemes:
    PasswordBearer:
      type: http
      scheme: bearer
      bearerFormat: Bearer ACCESS_TOKEN
      description: |
        This flow uses login and password to authenticate a user. Username and password are sent as parameters
        to the [/authorize](/rest-api-spec/#operation/authorize) endpoint.

        If the username and password are valid, then the broker issues an access token
        and returns it in the response. The access token is sent with every request in the Authorization header.

    OAuth2Bearer:
      type: oauth2
      flows:
        implicit:
          authorizationUrl: https://your-rest-implementation.com/api/authorize
          scopes:
            general: permission to perform all requests
        authorizationCode:
          authorizationUrl: https://your-rest-implementation.com/api/authorize
          tokenUrl: https://your-rest-implementation.com/api/oauth/token
          scopes:
            general: permission to perform all requests
      description: |
        TradingView is using the Implicit or Code flow of OAuth 2 as it is described in the [RFC 6749](https://tools.ietf.org/html/rfc6749).
        After performing authorization, the access token is sent with every request in the `Authorization` header.

        **Implicit Flow:**

        For OAuth 2 Implicit Flow the broker provides TradingView with the following OAuth 2 application parameters:

        Application parameter | Description | Example value
        -----------------|-------------|----------------------------------------
        client id        | A unique client identifier | sQeww4FgtythHTUrhrfMo9rRTq |
        authorization URI| An authorization endpoint URI | https://your-rest-implementation.com/api/authorize |
        scope            | A scope of the access request as a list of space-delimited, case-sensitive strings | read trade profile |

        TradingView provides the broker with two redirect URIs, one for the staging environment and the other for the production environment.

        *Authorization Request*

        TradingView sends the authorization request according to
        [OAuth 2 specification, 4.2.1](https://tools.ietf.org/html/rfc6749#section-4.2.1).
        The following parameters will be in the query part of the authorization URI:

        OAuth2 parameter | Description
        -----------------|------------
        response_type    | Response type. The value will be set to `token` |
        client_id        | A client identifier string provided by the broker |
        redirect_uri     | [Redirection Endpoint](https://tools.ietf.org/html/rfc6749#section-3.1.2) |
        scope            | A scope of the access request if it is provided by the broker |
        prompt           | The value will be set to `login` |
        state            | [A literal string](https://tools.ietf.org/html/rfc6749#section-4.1.1) that the broker should return to TradingView without changes in the authorization response.

        **Note**: The `redirect_uri` parameter should be verified by the broker's OAuth service, and should be whitelisted on the broker's service.

        TradingView could send the `lang` parameter additionally if it is required by the broker, with the following possible values:
        `ar`, `br`, `cs`, `de`, `el`, `en`, `es`, `fa`, `fr`, `he` ,`hu`, `id`, `in`, `it`, `ja`, `kr`, `ms`, `nl`, `pl`, `ro`, `ru`, `sv`, `th`, `tr`, `uk`, `vi`, `zh`.

        In case of successful authorization, the OAuth service redirects the user-agent (browser) to `redirect_uri`
        with the access token added as a fragment, according to
        [OAuth 2 specification, 4.2.2](https://tools.ietf.org/html/rfc6749#section-4.2.2). The `state` parameter
        should be passed without modifications. The `expires_in` parameter is optional, but is recommended to provide
        a more secure environment for integration users.

        *Refresh Request*

        In order to refresh the access token the broker should support additional prompt parameter in the authorization
        requests, that is not a part of the OAuth specification, but it presents in
        the [OpenID Connect specification](https://openid.net/specs/openid-connect-basic-1_0.html#RequestParameters).
        The broker should respect the `none` value of this parameter and display no authentication UI and just return
        an error if the user session is already expired (the specific error codes is also standardized by
        the OpenID Connect specification).

        **Code Flow:**

        For OAuth 2 Code Flow the broker provides TradingView with the following OAuth 2 application parameters:

        Application parameter | Description | Example value
        ----------------------|-------------|----------------------------------------
        client id             | A unique client identifier | sQeww4FgtythHTUrhrfMo9rRTq |
        client secret         | A client secret | 45dd67oi56ds678st6788dffdyu |
        authorization URI     | An authorization endpoint URI |https://your-rest-implementation.com/oauth/authorize |
        token URI             | A token endpoint URI | https://your-rest-implementation.com/oauth/token
        scope                 | A scope of the access request as a list of space-delimited, case-sensitive strings | read trade profile |

        TradingView provides the broker with two redirect URIs, one for the staging environment and the other for the production environment.

        *Authorization Request*

        TradingView sends the Authorization Request according to [OAuth 2 specification, 4.1.1](https://tools.ietf.org/html/rfc6749#section-4.1.1).
        The following parameters will be in the query part of the authorization URI:

        OAuth2 parameter | Description
        -----------------|------------
        response_type    | Response type. The value will be set to `code` |
        client_id        | A client identifier string provided by the broker |
        redirect_uri     | [Redirection Endpoint](https://tools.ietf.org/html/rfc6749#section-3.1.2) |
        scope            | A scope of the access request if it is provided by the broker |
        state            | [A literal string](https://tools.ietf.org/html/rfc6749#section-4.1.1) that the broker should return back to TradingView without changes in the authorization response

        **Note**: The `redirect_uri` parameter should be verified by the broker's OAuth service, and should be whitelisted on the broker's service.

        TradingView could send the `lang` parameter additionally if it is required by the broker, with the following possible values:
        `ar`, `br`, `cs`, `de`, `el`, `en`, `es`, `fa`, `fr`, `he` ,`hu`, `id`, `in`, `it`, `ja`, `kr`, `ms`, `nl`, `pl`, `ro`, `ru`, `sv`, `th`, `tr`, `uk`, `vi`, `zh`.

        If all the parameters of the authorization request are valid and access is granted, the broker issues an authorization code and redirects to the redirect URI
        with the following parameters added to the request URI component:

        OAuth2 parameter | Description
        -----------------|------------
        code  | The authorization code. It must expire shortly after its release to prevent the risk of leakage |
        state | The exact value which was sent in the authorization request |

        *Access Token Request*

        TradingView makes an Access Token Request by sending the following parameters using the `application/x-www-form-urlencoded` format:

        OAuth2 parameter | Description
        -----------------|------------
        grant_type       | The grant type. The value will be set to `authorization_code` |
        client_id        | A client identifier string provided by the broker |
        client_secret    | A client secret string provided by the broker |
        code             | The authorization code received from the broker |
        redirect_uri     | The same redirect URI used in the Authorization Request |

        After successful verification of the authorization code broker issues an access token,
        a refresh token and responds to the access token request as follows:

        OAuth2 parameter         | Description
        -------------------------|------------
        access_token             | The access token which will be used in every request to the API |
        refresh_token            | Token used to obtain new access tokens |
        token_type               | Token type. Should be set to `bearer` |
        expires_in               | The life time of the access token in seconds |
        refresh_token_expires_in | The life time of the refresh token in seconds |

        The `expires_in` and `refresh_token_expires_in` parameters are optional, but are recommended to provide
        a more secure environment for integration users.

        *Refresh Request*

        To refresh the access token TradingView makes a Refresh Request to the token endpoint using the `application/x-www-form-urlencoded` format with the following parameters:

        OAuth2 parameter | Description
        -----------------|------------
        grant_type       | The grant type. The value will be set to `refresh_token` |
        refresh_token    | The refresh token issued by the broker |
        client_secret    | The client secret value provided by the broker |

        If all parameters are valid, the broker issues a new access token and, if necessary,
        a new refresh token and responds to the request as described in the *Access token request* section above.

    ServerOAuth2Bearer:
      type: oauth2
      flows:
        clientCredentials:
          tokenUrl: https://your-rest-implementation.com/api/authorize/token
          scopes:
            general: permission to perform all requests
      description: |
        This authentication flow is used for server-to-server requests where authentication needs to be done without human intervention.
        It can be used with the [/history](#operation/getHistory), [/symbol_info](#operation/getSymbolInfo), [/streaming](#operation/streaming)
        and [/groups](#operation/getGroups) endpoints.

        JSON Web Token (JWT) is used to securely transmit information between broker and TradingView.
        More information about JWT can be found [here](https://jwt.io/introduction/).

        ### Application and OAuth2 parameters

        The broker provides the following application and OAuth2 parameters to TradingView:

        Application parameter | Description | Example value
        -----------------|-------------|---------------
        AppUrl | A URL uniquely representing broker's application | https://your-rest-implementation.com
        UserId | A user identifier for which the access token will be issued | testuser1

        OAuth2 parameter | Description | Example value
        -----------------|-------------|---------------
        client_id | A unique client Identifier | FGGH7-TRVC3-ECDD2-CDHRG3-FYUDG5 |
        client_secret | Secret code | adfegywrtyw |
        token_url | An url to which the access token request will be sent | https://your-rest-implementation.com/authorize/token
        grant_type | Indicates the type of grant being presented in exchange for an access token  | urn:your-broker:oauth:grant-type:personal-jwt

        ### X.509 Certificate
        The broker provides a valid X.509 certificate to TradingView for signing JWT.


        ### JSON Web Token
        JWT for the access token request is created with the following headers and claims:

        Header | Description | Standard | Sample Value
        -------|-----------|------------|---------------
        x5t | Thumbprint of provided X.509 certificate used for signing JWT | [RFC 7515](https://tools.ietf.org/html/rfc7515#section-4.1.7) | 935093f1690900asdfd98626dfw35fa22b41d9dfd
        alg | Algorithm used to sign JWT. RS256 is only supported at the moment | [RFC 1518](https://tools.ietf.org/html/rfc7518#page-6) | RS256

        Claim | Description | Sample Value
        ------|-------------|-------------
        iss | Issuer - Value should be client_id | FGGH7-TRVC3-ECDD2-CDHRG3
        sub | UserId - Value should be the UserId for which token is needed | testuser1
        exp | Expiry - Value should be a Unix time stamp indicating expiry of the token | 1549465914
        aud | Audience - Value should be the token_url | https://your-rest-implementation.com/authorize/token
        spurl | AppUrl - The AppUrl of your application | https://your-rest-implementation.com

        ### Requesting the access token
        TradingView sends a POST request to the `token_url` to obtain an access token.
        The `client_id` and `client_secret` are included as parameters in the request body.
        The JWT is sent in a parameter named `assertion`.

        #### Example Access Token request

            POST /token HTTP/1.1
            Host: sim.logonvalidation.net
            Content-Type: application/x-www-form-urlencoded

            grant_type=urn%3Ayour-broker%3Aoauth%3Agrant-type%3Apersonal-jwt
            &assertion=eyJhbGciOiJFUzI1NiIsImtpZCI6IjE2In0.eyJpc3Mi[...omitted for brevity...]
            &client_id=1234-5678-9101
            &client_secret=abcdefghijklmn

        The response to this request should contain an `access_token`. A `refresh_token` is optional and can be omitted:

        #### Example Access Token response
            {
              "access_token" : "eyJhbfGujudUf.eyJvYWEiOiIw[omitted for brevity]",
              "expires_in": 1200,
              "token_type": "Bearer",
              "refresh_token": "86fy7fa3d2-5e13-4736-86031-9ehiyiufrow0b",
              "refresh_token_expires_in": 2400
            }
  parameters:
    confirmId:
      name: confirmId
      in: query
      description: Identifier of an order received in the preview order request.
      schema:
        type: string
      required: false

    countback:
      name: countback
      in: query
      description: |
        Number of bars (higher priority than `from`) starting with `to`. If
        `countback` is set, `from` should be ignored.
      deprecated: true
      schema:
        type: number

    group:
      name: group
      in: query
      description: |
        ID of a symbol group. If it presents then only symbols that belong to this group should be returned.
        Possible values are provided by the [/groups](#operation/getGroups) endpoint.
        It is only required if you use groups of symbols to restrict access to instrument's data.
      schema:
        type: string

    historyFrom:
      name: from
      in: query
      description: 'Unix timestamp (UTC) of the leftmost required bar, including `from`.'
      schema:
        type: number
      required: true

    historyTo:
      name: to
      in: query
      description: |
        Unix timestamp (UTC) of the rightmost required bar, including `to`. It can be in the future.
        In this case, the rightmost required bar is the latest available bar.
      schema:
        type: number
      required: true

    locale:
      name: locale
      in: query
      required: true
      description: Locale (language) id.
      schema:
        $ref: '#/components/schemas/Locale'

    orderId:
      name: orderId
      description: Order identifier.
      in: path
      schema:
        type: string
      required: true

    pathAccountId:
      name: accountId
      in: path
      description: Account identifier.
      schema:
        type: string
      required: true

    positionId:
      name: positionId
      description: Position identifier.
      in: path
      schema:
        type: string
      required: true

    netPositionId:
      name: netPositionId
      description: Net position identifier.
      in: path
      schema:
        type: string
      required: true

    individualPositionId:
      name: individualPositionId
      description: Individual position identifier.
      in: path
      schema:
        type: string
      required: true

    requestId:
      name: requestId
      in: query
      description: Unique identifier for a request.
      schema:
        type: string
      required: false
      example: 23425678343

    resolution:
      name: resolution
      in: query
      description: |
        Symbol resolution. Possible resolutions are daily (`D` or `1D`, `2D` ... ),
        weekly (`1W`, `2W` ...), monthly (`1M`, `2M`...) and an intra-day
        resolution &ndash; minutes(`1`, `2` ...).
      schema:
        type: string
      required: true

    queryAccountId:
      name: accountId
      in: query
      description: Account identifier.
      schema:
        type: string
      required: true

    symbol:
      name: symbol
      in: query
      description: Symbol name or ticker.
      schema:
        type: string
      required: true
  schemas:
    AccessToken:
      title: AccessToken
      type: object
      description: Access token.
      properties:
        access_token:
          type: string
          description: |
            Access token acts as a session ID that the application uses for making
            requests. This token should be protected as if it were user
            credentials.
          example: '7133au-cba5a72-842029c'
        expiration:
          type: number
          description: |
            The time when the token is expired is represented as the number of
            seconds since the Unix epoch (00:00:00 UTC on 1 January 1970).
          example: 1548661401
      required:
        - access_token
        - expiration
      additionalProperties: false

    Account:
      title: Account
      type: object
      properties:
        id:
          description: Unique account identifier.
          type: string
          example: 'ACC-001'
        name:
          description: Account title that is displayed to a user.
          type: string
          example: 'Demo trading account'
        type:
          description: Account type
          type: string
          enum:
            - live
            - demo
          example: 'demo'
        currency:
          description: Abbreviation of account currency.
          type: string
          example: 'JPY'
        currencySign:
          description: Account currency symbol.
          type: string
          example: 'Â¥'
        config:
          $ref: '#/components/schemas/AccountFlags'
        ui:
          description: |
            Account manager and Order dialog configuration for the account.
            The Account manager configuration will override configuration, specified in the [/config](#operation/getConfiguration) endpoint.
          type: object
          properties:
            accountSummaryRow:
              $ref: '#/components/schemas/AccountSummaryRow'
            accountManager:
              $ref: '#/components/schemas/AccountManager'
            orderCustomFields:
              $ref: '#/components/schemas/OrderCustomFields'
            orderHistoryCustomFields:
              $ref: '#/components/schemas/OrderHistoryCustomFields'
            positionCustomFields:
              $ref: '#/components/schemas/PositionCustomFields'
            netPositionCustomFields:
              $ref: '#/components/schemas/PositionCustomFields'
            individualPositionCustomFields:
              $ref: '#/components/schemas/PositionCustomFields'
            orderDialogCustomFields:
              $ref: '#/components/schemas/OrderDialogCustomFields'
          additionalProperties: false
        durations:
          $ref: '#/components/schemas/AccountDurations'
        prefix:
          description: Prefix for instruments.
          type: string
          example: 'NASDAQ'
        isVerified:
          type: boolean
          default: false
          description: Used to confirm that the account has been verified (for example, KYC is passed). Only verified
            account users can leave reviews in the broker profile.
          example: true
      required:
        - id
        - name
        - type
        - config
      additionalProperties: false

    AccountDurations:
      title: AccountDurations
      type: array
      description: |
        Localized array of durations displayed in Order Ticket.
        It will override values, specified in the [/config](#operation/getConfiguration) endpoint.
      items:
        $ref: '#/components/schemas/Duration'
      minItems: 1

    AccountFlags:
      title: AccountFlags
      type: object
      properties:
        supportBrackets:
          description: Whether the integration supports brackets. Deprecated. Use supportOrderBrackets and
            supportPositionBrackets instead.
          type: boolean
          deprecated: true
        supportOrderBrackets:
          description: Whether the integration supports brackets (take profit and stop loss) for orders.
          type: boolean
          default: false
        supportMarketBrackets:
          description: Whether the integration supports brackets for market orders.
          type: boolean
          default: true
        supportPositionBrackets:
          description: Whether the integration supports adding (or modifying) stop loss and take profit to positions.
          type: boolean
          default: false
        supportPositions:
          description: Whether the integration supports the Positions tab. If you set it to `false`, the `/positions`
            endpoint will not be used.
          type: boolean
          default: true
        supportMultiposition:
          description: Whether the integration supports multiple positions at one instrument at the same time.
          type: boolean
          default: false
        supportClosePosition:
          description: Whether the integration supports closing of a position without a need for a user to fill an order.
          type: boolean
          default: false
        supportPartialClosePosition:
          description: Whether the integration supports partial closing of a position.
          type: boolean
          default: false
        supportReversePosition:
          description: Whether the integration supports reversing of a position. If this flag is set to `false`
            the reverse position button will be hidden.
          type: boolean
          default: true
        supportNativeReversePosition:
          description: Whether the integration natively supports reversing of a position. If it is natively supported
            then TradingView will send a request to the [Modify Position endpoint](#operation/modifyPosition) with
            the `side` parameter set. If it is not natively supported then a reversing order will be placed.
          type: boolean
          default: false
        supportMarketOrders:
          description: Whether the integration supports market orders.
          type: boolean
          default: true
        supportLimitOrders:
          description: Whether the integration supports limit orders.
          type: boolean
          default: true
        supportStopOrders:
          description: Whether the integration supports stop orders.
          type: boolean
          default: true
        supportStopLimitOrders:
          description: Whether the integration supports StopLimit orders.
          type: boolean
          default: false
        supportTrailingStop:
          description: Whether the integration supports trailing stop orders.
          type: boolean
          default: false
        supportStopOrdersInBothDirections:
          description: Whether Stop orders should behave like Market-if-touched in both directions. Enabling this flag
            removes the direction check from the order dialog.
          type: boolean
          default: false
        supportStopLimitOrdersInBothDirections:
          description: Whether StopLimit orders should behave like Limit-if-touched in both directions. Enabling this flag
            removes the direction check from the order dialog.
          type: boolean
          default: false
        supportPartialOrderExecution:
          description: Whether the integration supports partial order's execution. If this flag is set to `true`, then
            the 'Filled Qty' column will be displayed on the Orders tab.
          type: boolean
          default: false
        supportModifyOrder:
          description: Whether the integration supports the modification of the existing order. Deprecated. Use
            supportModifyOrderPrice, supportEditAmount and supportModifyBrackets instead.
          type: boolean
          deprecated: true
        supportModifyOrderPrice:
          description: Whether the integration supports order price editing. If you set it to `false`, the price control
            in the order ticket will be disabled when modifying an order.
          type: boolean
          default: true
        supportEditAmount:
          description: Whether the integration supports editing orders quantity. If you set it to `false`, the quantity
            control in the order ticket will be disabled when modifying an order.
          type: boolean
          default: true
        supportModifyBrackets:
          description: Whether the integration supports order brackets editing. If you set it to `false`, the bracket's
            control in the order ticket will be disabled when modifying an order, and 'Modify' button will be hidden on
            a chart and in the Account Manager.
          type: boolean
          default: true
        supportModifyDuration:
          description: Whether the integration supports the modification of the duration of the existing order.
          type: boolean
          default: false
        supportCryptoExchangeOrderTicket:
          description: Whether the account is used to exchange(trade) crypto currencies. This flag switches
            the Order Ticket to the Crypto Exchange mode. It adds second currency quantity control, currency labels etc.
          type: boolean
          default: false
        supportDigitalSignature:
          description: Whether the integration supports Digital signature input field in the Order Ticket.
          type: boolean
          default: false
        supportPlaceOrderPreview:
          description: Whether the integration supports providing and displaying information (such as commission,
            margin, value, etc.) about the order being placed before submitting it.
          type: boolean
          default: false
        supportModifyOrderPreview:
          description: Whether the integration supports providing and displaying information (such as commission,
            margin, value, etc.) about the order being modified before submitting it.
          type: boolean
          default: false
        showQuantityInsteadOfAmount:
          description: Renames Amount to Quantity in the Order Ticket.
          type: boolean
          default: false
        supportBalances:
          description: Whether the integration supports [/balances](/rest-api-spec/#operation/getBalances) request.
          type: boolean
          default: false
        supportOrdersHistory:
          description: Whether the integration supports [/ordersHistory](/rest-api-spec/#operation/getOrdersHistory)
            request.
          type: boolean
          default: false
        supportExecutions:
          description: Whether the integration supports [/executions](/rest-api-spec/#operation/getExecutions) request.
          type: boolean
          default: false
        supportLeverage:
          description: Whether the integration supports leverage. If the flag is set to `true`, broker will calculate
            leverage using [/getLeverage](/rest-api-spec/#operation/getLeverage) endpoint.
          type: boolean
          default: false
        supportLeverageButton:
          description: Whether the integration supports leverage button. If the flag is set to `true`, a leverage input field
            will appear in the Order Widget. Click on the input field will activate a dedicated Leverage Dialog.
            Please note, that the flag `supportLeverage` should be enabled for the button to display.
          type: boolean
          default: true
        supportDOM:
          description: Whether the integration supports DOM (Depth of market) widget to be available.
          type: boolean
          default: true
        supportLevel2Data:
          description: Whether the integration supports Level 2 data. It is required to display DOM levels. You must
            implement [/depth](/rest-api-spec/#operation/getDepth) endpoint to display DOM.
          type: boolean
          default: false
        supportPLUpdate:
          description: Whether the integration provide `unrealizedPl` for positions. Otherwise P&L will be calculated
            automatically based on a simple algorithm.
          type: boolean
          default: true
        supportDisplayBrokerNameInSymbolSearch:
          description: Whether the integration involves displaying broker instrument names in the Symbol Search. You
            may usually want to disable it if broker symbols are the same or you are using internal numbers as broker
            symbol names.
          type: boolean
          default: true
        supportLogout:
          description: Whether the integration supports logout.
          type: boolean
          default: false
        supportCustomAccountSummaryRow:
          description: Whether the integration supports custom Account Summary Row.
          type: boolean
          default: false
        supportPositionCustomFields:
          description: Whether the integration supports custom fields for position.
          type: boolean
          default: false
        supportOrderCustomFields:
          description: Whether the integration supports custom fields for order.
          type: boolean
          default: false
        supportOrderHistoryCustomFields:
          description: Whether the integration supports custom fields for order history.
          type: boolean
          default: false
        supportTrackLatency:
          description: Whether the integration supports latency tracking.
            See the [/trackLatency](#operation/trackLatency) endpoint.
          type: boolean
          default: false
        supportPositionNetting:
          description: |
            Whether the integration supports individual and net positions.
            Instead of requesting positions from the [/positions](#operation/getPositions) endpoint, the client will request individual and net positions separately.
            'Positions' tab in the Account Manager will be divided into two subcategories: 'Individual Positions' and 'Net Positions'.

            Enables the [/individualPositions](#operation/getIndividualPositions) and [/netPositions](#operation/getNetPositions) endpoints
            as well as [/modifyIndividualPosition](#operation/modifyIndividualPosition), [/closeIndividualPosition](#operation/closeIndividualPosition), [/closeNetPosition](#operation/closeNetPosition).

            Disables the [/positions](#operation/getPositions), [/modifyPosition](#operation/modifyPosition) and [/closePosition](#operation/closePosition) endpoints.
          type: boolean
          default: false
        supportIndividualPositionBrackets:
          description: Whether the integration supports adding (or modifying) stop loss and take profit to individual positions.
          type: boolean
          default: false
        supportPartialCloseIndividualPosition:
          description: Whether the integration supports partial closing of an individual position.
          type: boolean
          default: false
      additionalProperties: false

    AccountManager:
      title: AccountManager
      type: array
      description: |
        Localized Account manager's tables configuration. Account manager is a
        page in the bottom widget. This page can have multiple tables. Data of
        the tables is filled using the [/state](/rest-api-spec/#operation/getState) endpoint.
      items:
        $ref: '#/components/schemas/AccountManagerTable'
      minItems: 1
      example: [
        {
          "id": "accountSummary",
          "title": "",
          "columns": [
            {
              "id": "todayPL",
              "title": "Today's P&L",
            },
            {
              "id": "accountValue",
              "title": "Account Value",
            },
            {
              "id": "balance",
              "title": "Balance",
            },
            {
              "id": "totalMargin",
              "title": "Margin",
            },
            {
              "id": "held",
              "title": "Held",
            },
            {
              "id": "buyingPower",
              "title": "Buying Power",
            }
          ]
        }
      ]

    AccountManagerColumn:
      title: AccountManagerColumn
      type: object
      properties:
        id:
          type: string
          example: 'balance'
        title:
          description: Localized title of a column.
          type: string
          example: 'Balance'
        tooltip:
          description: Tooltip that is shown on mouse hover.
          type: string
          example: 'Balance column'
        alignment:
          description: The cell value alignment. Default value is `left`
          type: string
          enum:
            - left
            - right
          example: 'right'
          default: 'left'
        isCapitalize:
          description: The first character of each word in the sentence will be capitalized. The rest of the symbols
            appearance does not change.
          type: boolean
          default: true
      required:
        - id
        - title
      additionalProperties: false

    AccountManagerTable:
      title: AccountManagerTable
      type: object
      properties:
        id:
          type: string
          example: "accountSummary"
        title:
          description: Localized title of a table.
          type: string
          example: 'Account Summary'
        columns:
          type: array
          items:
            $ref: '#/components/schemas/AccountManagerColumn'
          minItems: 1
      required:
        - id
        - columns
      additionalProperties: false

    AccountResponse:
      title: AccountResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/Account'
          minItems: 1
      required:
        - s
        - d
      additionalProperties: false

    AccountState:
      type: object
      properties:
        balance:
          description: Account Balance.
          type: number
        unrealizedPl:
          description: Unrealized profit/loss.
          type: number
        equity:
          description: Equity. If this field is not specified, then it is calculated as balance + unrealizedPl.
          type: number
        amData:
          description: |
            Account manager data. Structure of Account manager is defined by the
            [/config](#operation/getConfiguration) endpoint. Each element of this array is a table.
          type: array
          items:
            description: |
              Single Account manager table data. Each element of this array is a table row.
            type: array
            items:
              description: |
                Account manager table rows data. Each element of this array is a table cell.
              type: array
              items:
                type: string
          minItems: 1
        accountSummaryRowData:
          description: |
            Account Summary Row data. Structure of Account Summary Row is defined by the
            [/config](#operation/getConfiguration) endpoint.
          type: array
          items:
            type: string
          minItems: 1
        marginAvailable:
          description: Current available margin for account.
          type: number
      required:
        - balance
        - unrealizedPl
      additionalProperties: false
      example: {
        "balance": 41757.91,
        "unrealizedPl": 1053.02,
        "equity": 42857.56,
        "amData": [
          [
            [
                "90.22",
                "42857.56",
                "42857.56",
                "1099.65",
                "0.00",
                "41757.91"
            ]
          ]
        ],
        "accountSummaryRowData": [
            "96883.89",
            "96837.12",
            "-46.77"
        ]
      }

    AccountStateResponse:
      title: AccountStateResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/AccountState'
      required:
        - s
        - d
      additionalProperties: false

    AccountStateMessage:
      title: AccountStateMessage
      $ref: '#/components/schemas/AccountState'
      additionalProperties: false

    AccountSummaryRow:
      title: AccountSummaryRow
      type: array
      description:
        Localized Account Summary Row configuration.

        Account Summary Row is a panel at the top of the bottom widget with the key account indicators.
        The `supportCustomAccountSummaryRow` flag should be enabled in the account config,
        see the [/accounts](#operation/getAccounts) endpoint.
        Data of the Account Summary Row is filled using the [/state](/rest-api-spec/#operation/getState) endpoint.
      items:
        $ref: '#/components/schemas/AccountSummaryRowItem'
      minItems: 1
      example:
        [
          {
            "id": "accountBalance",
            "title": "Account Balance"
          },
          {
            "id": "Equity",
            "title": "Realized P/L"
          },
          {
            "id": "Open Profit",
            "title": "Unrealized P/L"
          }
        ]

    AccountSummaryRowItem:
      title: AccountSummaryRowItem
      type: object
      properties:
        id:
          type: string
          description: Unique Account Summary Row item identifier.
        title:
          type: string
          description: Localized Account Summary Row item title.
      required:
        - id
        - title
      additionalProperties: false

    AuthorizeResponse:
      title: AuthorizeResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/AccessToken'
      required:
        - s
        - d
      additionalProperties: false

    BarsArrays:
      title: BarsArrays
      type: object
      description: Bars data.
      properties:
        t:
          description: |
            Bar time, Unix timestamp (UTC). Daily bars should only have the date
            part, time should be 0.
          type: array
          items:
            type: number
          example: [1547942400, 1547942460,  1547942520]
        o:
          description: Open price.
          type: array
          items:
            type: number
          example: [3667, 3662.25, 3664.29]
        h:
          description: High price.
          type: array
          items:
            type: number
          example: [3667.24, 3664.47, 3664.3]
        l:
          description: Low price.
          type: array
          items:
            type: number
          example: [3661.55, 3661.9, 3662.43]
        c:
          description: Close price.
          type: array
          items:
            type: number
          example: [3662.25, 3663.13, 3664.01]
        v:
          description: Volume.
          type: array
          items:
            type: number
          example: [34.7336, 2.4413, 11.7075]
      required:
        - t
        - o
        - h
        - l
        - c
        - v

    CheckBoxMetainfo:
      title: CheckBoxMetainfo
      type: object
      properties:
        id:
          description: Unique field identifier.
          type: string
          example: customInputFieldId
        title:
          description: Localized checkbox display name.
          type: string
          example: Additional parameter
        saveToSettings:
          description: Whether the value should be stored in the user settings and preserved for the next time
            the dialog is displayed.
          type: boolean
          default: false
          example: true
        help:
          description: Help text displayed when hovering over the checkbox.
          type: string
          example: Brief help text
        checked:
          description: Initial checkbox value.
          type: boolean
          default: false
          example: false
        mutable:
          description: Whether this field can be changed during the order modification.
          type: boolean
          default: true
          example: false
      required:
        - id
        - title

    ComboBoxMetaInfo:
      title: ComboBoxMetaInfo
      type: object
      properties:
        id:
          description: Unique field identifier.
          type: string
          example: customInputFieldId
        title:
          description: Localized field display name.
          type: string
          example: Additional parameters
        saveToSettings:
          description: Whether the value should be stored in the user settings and preserved for the next time
            the dialog is displayed.
          default: false
          type: boolean
          example: true
        mutable:
          description: Whether this field can be changed during the order modification.
          type: boolean
          default: true
          example: false
        forceUserEnterInitialValue:
          description: If this flag is set to true, the user will not be able to place an order without explicitly
            entering a value, so instant order placement is not available.
          type: boolean
          default: false
          example: true
        items:
          type: array
          items:
            $ref: '#/components/schemas/ComboBoxValue'
          minItems: 1
      required:
        - id
        - items
        - title
      additionalProperties: false

    ComboBoxValue:
      title: ComboBoxValue
      type: object
      properties:
        text:
          description: Localized display name.
          example: text
          type: string
        value:
          description: Value that is sent back to the server if the item is selected.
          example: value
          type: string
      required:
        - text
        - value
      additionalProperties: false

    Config:
      title: Config
      type: object
      properties:
        accountSummaryRow:
          $ref: '#/components/schemas/AccountSummaryRow'
        accountManager:
          $ref: '#/components/schemas/AccountManager'
        durations:
          $ref: '#/components/schemas/Durations'
        orderCustomFields:
          $ref: '#/components/schemas/OrderCustomFields'
        orderHistoryCustomFields:
          $ref: '#/components/schemas/OrderHistoryCustomFields'
        positionCustomFields:
          $ref: '#/components/schemas/PositionCustomFields'
        netPositionCustomFields:
          $ref: '#/components/schemas/PositionCustomFields'
        individualPositionCustomFields:
          $ref: '#/components/schemas/PositionCustomFields'
        pullingInterval:
          $ref: '#/components/schemas/PullingInterval'
      additionalProperties: false

    ConfigResponse:
      title: ConfigResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/Config'
      required:
        - s
        - d
      additionalProperties: false

    CryptoBalance:
      title: CryptoBalance
      type: object
      properties:
        symbol:
          type: string
          description: Crypto currency symbol.
          example: 'BTC'
        longName:
          type: string
          description: Crypto currency name.
          example: 'Bitcoin'
        total:
          type: number
          description: Total amount of the balance.
          example: 1000
        available:
          type: number
          description: The balance available to the user.
          example: 10
        reserved:
          type: number
          description: Reserved balance amount that can't be used at the moment for any reasons.
          example: 90
        btcValue:
          type: number
          description: Total balance amount in BTC.
          example: 1000
        value:
          type: number
          description: Balance amount in additional currency.
          example: 100000
        valueCurrency:
          type: string
          description: Currency of the value. It can be either code, or symbol.
          example: 'USD'
      required:
        - symbol
        - total
        - available
      additionalProperties: false

    CryptoBalancesResponse:
      title: CryptoBalancesResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/CryptoBalance'
      required:
        - s
        - d
      additionalProperties: false

    CustomFieldsValueItem:
      title: CustomFieldsValueItem
      type: object
      properties:
        id:
          description: Column identifier as defined in [/config](#operation/getConfiguration) response.
          type: string
        value:
          description: Localized column value
          type: string
      required:
        - id
        - value
      additionalProperties: false

    Depth:
      title: Depth
      type: object
      description: Depth of market for an instrument.
      properties:
        asks:
          description: Array of arrays with two numeric elements - price and volume. Must be sorted by `price` in ascending order.
          type: array
          items:
            $ref: '#/components/schemas/DepthItem'
        bids:
          description: Array of arrays with two numeric elements - price and volume. Must be sorted by `price` in ascending order.
          type: array
          items:
            $ref: '#/components/schemas/DepthItem'
      required:
        - asks
        - bids
      additionalProperties: false
      example: {asks: [[45.10, 100], [48.4, 120]], bids: [[24.7, 80], [35.6, 30]]}

    DepthItem:
      title: DepthItem
      type: array
      description: Array with two numeric elements - price and volume.
      items:
        type: number
      minItems: 1

    DepthResponse:
      title: DepthResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/Depth'
      required:
        - s
        - d
      additionalProperties: false

    Duration:
      title: Duration
      description: Single duration option.
      type: object
      properties:
        id:
          description: Duration identifier.
          type: string
          example: 'GTT'
        title:
          description: Localized title.
          type: string
          example: 'Good Till Time'
        hasDatePicker:
          description: Display date control in Order Ticket for this duration type.
          type: boolean
          default: false
          example: true
        hasTimePicker:
          description: Display time control in Order Ticket for this duration type.
          type: boolean
          default: false
          example: true
        default:
          description: Default duration. Only one duration object in the durations array can have a `true` value for
            this field. The default duration will be used when the user places orders in the silent mode and it will be
            the selected one when the user opens the Order dialog for the first time.
          type: boolean
          default: false
          example: true
        supportedOrderTypes:
          description: An optional array of order types to which the duration will be applied.
          type: array
          items:
            type: string
            enum:
              - market
              - stop
              - limit
              - stoplimit
          minItems: 1
          default: ['limit', 'stop', 'stoplimit']
          example: ['stop', 'stoplimit']
      required:
        - id
        - title
      additionalProperties: false

    Durations:
      title: Durations
      type: array
      description: Localized array of durations displayed in Order Ticket.
      items:
        $ref: '#/components/schemas/Duration'
      minItems: 1

    EmptyBarArrays:
      title: EmptyBarArrays
      type: object
      description: Empty Bars data.
      properties:
        t:
          type: array
          items:
            type: number
          description: Empty array
          maxItems: 0
          example: []
        o:
          type: array
          items:
            type: number
          description: Empty array
          maxItems: 0
          example: []
        h:
          type: array
          items:
            type: number
          description: Empty array
          maxItems: 0
          example: []
        l:
          type: array
          items:
            type: number
          description: Empty array
          maxItems: 0
          example: []
        c:
          type: array
          items:
            type: number
          description: Empty array
          maxItems: 0
          example: []
        v:
          type: array
          items:
            type: number
          description: Empty array
          maxItems: 0
          example: []

    ErrorResponse:
      title: ErrorResponse
      $ref: '#/components/schemas/Error'
      additionalProperties: false

    Execution:
      title: Execution
      type: object
      properties:
        id:
          description: Unique identifier.
          type: string
          example: EX34567
        instrument:
          description: Instrument id.
          type: string
          example: 'EURUSD'
        price:
          description: Execution price.
          type: number
          example: 1.23564
        time:
          description: Execution time, Unix timestamp (UTC).
          type: number
          example: 1548406235
        qty:
          description: Execution quantity.
          type: number
          example: 1
        side:
          description: Side.
          type: string
          enum:
            - buy
            - sell
          example: 'buy'
        orderId:
          description: Identifier of the order that has been filled.
          type: string
          example: '1263159154'
        isClose:
          description: Whether the execution reduces the position.
          type: boolean
          example: false
        positionId:
          description: Identifier of the position that has been opened, modified or closed.
          type: string
          example: '10098'
        commission:
          description: Commission charged for the fill.
          type: number
          example: 0.004
      required:
        - id
        - instrument
        - price
        - time
        - qty
        - side
        - orderId
        - isClose
      additionalProperties: false

    ExecutionsResponse:
      title: ExecutionsResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/Execution'
      required:
        - s
        - d
      additionalProperties: false

    GetLeverage:
      title: GetLeverage
      type: object
      properties:
        title:
          description: Title for leverage adjustment popup.
          type: string
        leverage:
          description: Current leverage value.
          type: number
        min:
          description: Minimum leverage value.
          type: number
        max:
          description: Maximum leverage value.
          type: number
        step:
          description: Leverage value change step.
          type: number
      required:
        - title
        - leverage
        - min
        - max
        - step
      additionalProperties: false
      example: {
        "title": "Adjust Leverage",
        "leverage": 5,
        "min": 1,
        "max": 10,
        "step": 1
      }

    GetLeverageResponse:
      title: GetLeverageResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/GetLeverage'
      required:
        - s
        - d
      additionalProperties: false

    GroupList:
      title: GroupList
      type: object
      properties:
        groups:
          type: array
          description: |
            Each element of this array is an group object.
          items:
            type: object
            properties:
              id:
                description: |
                  All characters in a group id must be either a lowercase alphabetic character or an underscore.
                  A group id should start with the same prefix related to the broker's name.
                type: string
            required:
              - id
            additionalProperties: false
      additionalProperties: false
      example: {groups: [{id: 'broker_stocks'}, { id: 'broker_futures' }, {id: 'broker_forex'}, { id: 'broker_crypto' }]}

    GroupListResponse:
      title: GroupListResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/GroupList'
      required:
        - s
        - d
      additionalProperties: false

    HistoryEmptyBarResponse:
      title: HistoryEmptyBarResponse
      allOf:
        - $ref: '#/components/schemas/SuccessResponse'
        - $ref: '#/components/schemas/EmptyBarArrays'

    HistorySuccessResponse:
      title: HistorySuccessResponse
      allOf:
        - $ref: '#/components/schemas/SuccessResponse'
        - $ref: '#/components/schemas/BarsArrays'

    Instrument:
      title: Instrument
      type: object
      properties:
        name:
          description: Broker instrument name.
          type: string
          example: 'EURUSD'
        description:
          description: Instrument description.
          type: string
          example: 'EUR/USD'
        minQty:
          description: Minimum quantity for trading. If `lotSize` is set, then the specified value must be in lots.
          type: number
          example: 1
        maxQty:
          description: Maximum quantity for trading. If `lotSize` is set, then the specified value must be in lots.
          type: number
          example: 100000000
        qtyStep:
          description: Quantity step. If `lotSize` is set, then the specified value must be in lots.
          type: number
          example: 100
        pipSize:
          description: |
            Size of 1 pip.
            It is equal to `minTick` for non-forex symbols. For forex pairs it equals either the `minTick`,
            or the `minTick` multiplied by `10`. For example, for IBM `minTick` it is 0.01, for EURCAD `minTick` it is 0.00001.
          type: number
          example: 0.0001
        pipValue:
          description: |
            Value of 1 pip in the account currency.
          type: number
          example: 0.00008845
        minTick:
          description: Minimum price movement. For example, for IBM `minTick` is 0.01, for EURCAD `minTick` is 0.00001.
          type: number
          example: 0.00001
        lotSize:
          description: Financial instrument units standardized number as set by the exchange or broker for buying or selling.
          type: number
          example: 10
        baseCurrency:
          description: The first currency quoted in a currency pair. Used for crypto currencies only.
          type: string
          example: 'LTC'
        quoteCurrency:
          description: |
            A quote currency is the second currency quoted in a currency pair. Used for crypto currencies only.
          type: string
          example: 'BTC'
        marginRate:
          description: Margin rate for this instrument.
          type: number
          example: 0.05
        hasQuotes:
          type: boolean
          description: Indicates if your API provides quotes for this instrument. Assigning `false` to this field
            prevents `/quotes` request and makes ask/bid displayed from a TradingView server depending on users data
            subscriptions on TradingView. Use of this flag must be agreed with TradingView
          default: true
          example: false
        units:
          description: Units of quantity or amount. Displayed instead of the `Units` label in the Quantity/Amount field
          type: string
        variableMinTick:
          description:
            Dynamic minimum price movement. It is used if the instrument's minimum price movement changes depending on the price range. For example, '0.01 10 0.02 25 0.05',
            where `minTick` is 0.01 for a price less than 10, `minTick` is 0.02 for a price less than 25, `minTick` is 0.05 for a price more and equal than 25.
          type: string
          example: '0.01 10 0.02 25 0.05'
        type:
          $ref: '#/components/schemas/SymbolType'
        ui:
          description: |
            Order dialog configuration for the symbol. It will override configuration,
            specified in the [/accounts](#operation/getAccounts) endpoint.
          type: object
          properties:
            orderDialogCustomFields:
              $ref: '#/components/schemas/OrderDialogCustomFields'
            validationRules:
              description: |
                Rules applied to field validation in the Order dialog.
                Please note: `stopPercent` and `limitPercent` validation rules are not applied
                if the `supportStopOrdersInBothDirections` or `supportStopLimitOrdersInBothDirections` flag is set to `true`.
              type: array
              items:
                anyOf:
                  - $ref: '#/components/schemas/StopPercentValidationRule'
                  - $ref: '#/components/schemas/LimitPercentValidationRule'
                  - $ref: '#/components/schemas/StopLossPercentValidationRule'
                  - $ref: '#/components/schemas/TakeProfitPercentValidationRule'
              minItems: 1
              example: [{"id": "slPercent", "options": {"min": 10, "max": 90}}, {"id": "tpPercent", "options": {"min": 10, "max": 90}}, {"id": "limitPercent", "options": {"min": 10, "max": 20}}, {"id": "stopPercent", "options": {"min": 10, "max": 20}}]
          additionalProperties: false
      required:
        - name
        - description
        - pipSize
        - pipValue
        - minTick
        - type
      additionalProperties: false

    InstrumentsResponse:
      title: InstrumentsResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/Instrument'
      required:
        - s
        - d
      additionalProperties: false

    LimitPercentValidationRule:
      title: LimitPercentValidationRule
      description: Limit Percent Price Range Rule applied to the order price field validation in the Order dialog.
      type: object
      properties:
        id:
          type: string
          enum:
            - limitPercent
        options:
          type: object
          properties:
            min:
              description: Minimal value in percentage of the limit price relative to the current price.
              type: number
              example: 10
            max:
              description: Maximum value in percentage of the limit price relative to the current price.
              type: number
              example: 20
          required:
            - min
            - max
          additionalProperties: false
      required:
        - id
        - options
      additionalProperties: false

    Locale:
      title: Locale
      description: Locale (language) id. This parameter is required for trading integration only.
      type: string
      enum:
        - ar_AE
        - br
        - cs
        - de_DE
        - el
        - en
        - es
        - fa_IR
        - fr
        - he_IL
        - hu_HU
        - id
        - in
        - it
        - ja
        - kr
        - ms_MY
        - nl_NL
        - pl
        - ro
        - ru
        - sv_SE
        - th_TH
        - tr
        - uk
        - vi_VN
        - zh_CN
        - zh_TW

    MappingResponse:
      title: MappingResponse
      $ref: '#/components/schemas/SymbolMapping'

    Message:
      title: Message
      description: |
        Informational message description, that will be displayed for the order or position in the Account manager. The message will be marked
        with a color, corresponding to a message type.
        Also, the message text will be displayed in the notification pop-up in case that order type is set to `rejected`.
      type: object
      properties:
        text:
          description: Message text
          type: string
          example: "You can add brackets to this\nposition to protect it"
        type:
          description: 'Message type'
          type: string
          enum:
            - information
            - warning
            - error
          example: 'Information'
      required:
        - text
        - type
      additionalProperties: false

    OkStatus:
      title: OkStatus
      type: string
      description: Status will always be `ok`.
      enum:
        - ok
      example: 'ok'

    Order:
      title: Order
      allOf:
        - $ref: '#/components/schemas/OrderCommon'
        - $ref: '#/components/schemas/OrderStatus'

    OrderCommon:
      title: OrderCommon
      type: object
      properties:
        id:
          description: Unique identifier.
          type: string
        instrument:
          description: Instrument name that is used on a broker's side.
          type: string
        qty:
          description: Quantity.
          type: number
        side:
          description: Side.
          type: string
          enum:
            - buy
            - sell
        type:
          description: Type.
          type: string
          enum:
            - market
            - stop
            - limit
            - stoplimit
        filledQty:
          description: Filled quantity.
          type: number
        avgPrice:
          description: |
            Average price of order fills. It should be provided for filled / partly filled orders.
          type: number
        limitPrice:
          description: Limit Price for Limit or StopLimit orders only. You should not send this field for other order types.
          type: number
        stopPrice:
          description: Stop Price for Stop or StopLimit orders only. You should not send this field for other order types.
          type: number
        trailingStopPips:
          description: Distance from the stop loss level to the current market price in pips for a position or to the order price if the parent is a stop or limit order.
          type: number
        isTrailingStop:
          description: If this flag is set to `true`, then the stop order is a trailing stop.
          type: boolean
        parentId:
          description: |
            Identifier of a parent order or a parent position (for position brackets) depending on `parentType`.
            Should be set only for bracket orders.
          type: string
        parentType:
          description: Type of order's parent. Should be set only for bracket orders.
          type: string
          enum:
            - order
            - position
        duration:
          description: Expiration type and Unix timestamp date/time.
          type: object
          properties:
            type:
              description: Duration ID. Internal ID that you set in [/config](#operation/getConfiguration) response.
              type: string
            datetime:
              description: Unix timestamp (UTC).
              type: number
          required:
            - type
        lastModified:
          description: Indicates the time when the order was last modified, Unix timestamp (UTC).
          type: number
        customFields:
          type: array
          description: |
            Localized order custom fields values data.
            Custom fields are configured in the [/config](#operation/getConfiguration) endpoint response.
            Custom `Order dialog` fields are to be sent along with the standard fields in the order object.
          items:
            $ref: '#/components/schemas/CustomFieldsValueItem'
          minItems: 1
        message:
          $ref: '#/components/schemas/Message'
      required:
        - id
        - instrument
        - qty
        - side
        - type

    OrderCustomFields:
      title: OrderCustomFields
      type: array
      description: |
        Additional order columns to be displayed in the orders table of the Account manager.
        The orders table has a default set of columns that can be extended using this configuration.
        The `supportOrderCustomFields` flag should be enabled in the [account configuration](https://www.tradingview.com/rest-api-spec/#operation/getAccounts).
        Data of the additional fields is filled from the
        [/orders](https://www.tradingview.com/rest-api-spec/#operation/getOrders) `customFields` value.

      items:
        $ref: '#/components/schemas/AccountManagerColumn'
      minItems: 1
      example: [
        {
          "id": "commission",
          "title": "Commission",
          "tooltip": "Commission Fees",
          "alignment": "right"
        }
      ]

    OrderDialogCustomFields:
      title: OrderDialogCustomFields
      description: Additional custom controls to be displayed in the Order dialog. At the moment,
        the only possible control types are combobox and checkbox. Data of the additional fields is filled from
        the [/orders](#operation/getOrders) endpoint.
      type: object
      properties:
        comboBox:
          type: array
          items:
            $ref: '#/components/schemas/ComboBoxMetaInfo'
          minItems: 1
        checkbox:
          type: array
          items:
            $ref: '#/components/schemas/CheckBoxMetainfo'
      additionalProperties: false

    OrderHistory:
      title: OrderHistory
      allOf:
        - $ref: '#/components/schemas/OrderCommon'
        - $ref: '#/components/schemas/OrderHistoryStatus'

    OrderHistoryCustomFields:
      title: OrderHistoryCustomFields
      type: array
      description: |
        Additional history columns to be displayed in the history table of the Account manager.
        The history table has a default set of columns that can be extended using this configuration.
        The `supportOrderHistoryCustomFields` and `supportOrdersHistory` flags should be enabled in the [account configuration](https://www.tradingview.com/rest-api-spec/#operation/getAccounts).
        Data of the additional fields is filled from the
        [/ordersHistory](/rest-api-spec/#operation/getOrdersHistory) `customFields` value.

      items:
        $ref: '#/components/schemas/AccountManagerColumn'
      minItems: 1
      example: [
        {
          "id": "commission",
          "title": "Commission",
          "tooltip": "Commission Fees",
          "alignment": "right"
        }
      ]

    OrderHistoryStatus:
      title: OrderHistoryStatus
      type: object
      properties:
        status:
          description: >
            String constants to describe a final order status.


            `Status`  | Description

            ----------|-------------

            rejected  | order is rejected for some reason

            filled    | order is fully executed

            cancelled  | order is cancelled
          type: string
          enum:
            - rejected
            - filled
            - cancelled
      required:
        - status

    OrdersHistoryResponse:
      title: OrdersHistoryResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/OrderHistory'
          example: [
            {
              "id": "1",
              "instrument": "EURUSD",
              "qty": 100,
              "side": "buy",
              "type": "limit",
              "avgPrice": 0,
              "limitPrice": 1.14344,
              "duration": {
                "type": "gtt",
                "datetime": 1548406235
              },
              "status": "filled"
            },
            {
              "id": "2",
              "instrument": "EURUSD",
              "qty": 100,
              "side": "sell",
              "type": "limit",
              "filledQty": 50,
              "avgPrice": 0,
              "limitPrice": 1.15094,
              "parentId": "1",
              "parentType": "order",
              "duration": {
                "type": "gtt",
                "datetime": 1548406235
              },
              "status": "cancelled"
            }
          ]
      required:
        - s
        - d
      additionalProperties: false

    OrderPreviewSection:
      title: OrderPreviewSection
      description: Describes a single order preview section. Order preview can have multiple sections that are divided by separators and may have titles.
      type: object
      properties:
        rows:
          description: Array of order preview items. Each item is a row of the section table.
          type: array
          items:
            $ref: '#/components/schemas/OrderPreviewSectionRow'
        header:
          description: Optional title of the section.
          type: string
      required:
        - rows
      additionalProperties: false

    OrderPreviewSectionRow:
      title: OrderPreviewSectionRow
      description: Describes a single row of a section table of the order preview.
      type: object
      properties:
        title:
          description: Description of the item.
          type: string
        value:
          description: Formatted value of the item.
          type: string
      required:
        - title
        - value
      additionalProperties: false

    OrdersResponse:
      title: OrdersResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/Order'
          example: [
            {
              "id": "1",
              "instrument": "EURUSD",
              "qty": 100,
              "side": "buy",
              "type": "limit",
              "avgPrice": 0,
              "limitPrice": 1.14344,
              "duration": {
                "type": "gtt",
                "datetime": 1548406235
              },
              "status": "working",
              "customFields": [{
                "id": "commission",
                "value": "1.25"
              }]
            },
            {
              "id": "2",
              "instrument": "EURUSD",
              "qty": 100,
              "side": "sell",
              "type": "limit",
              "filledQty": 50,
              "avgPrice": 0,
              "limitPrice": 1.15094,
              "parentId": "1",
              "parentType": "order",
              "duration": {
                "type": "gtt",
                "datetime": 1548406235
              },
              "status": "inactive",
              "customFields": [{
                "id": "commission",
                "value": ""
              }]
            },
            {
              "id": "3",
              "instrument": "EURUSD",
              "qty": 1000000,
              "side": "sell",
              "type": "limit",
              "filledQty": 0,
              "avgPrice": 0,
              "limitPrice": 1.15094,
              "parentId": "1",
              "parentType": "order",
              "duration": {
                "type": "gtt",
                "datetime": 1548406235
              },
              "status": "rejected",
              "message": {
                "text": "Insufficient Margin",
                "type": 'error'
              }

            }
          ]
      required:
        - s
        - d
      additionalProperties: false

    OrdersMessage:
      title: OrdersMessage
      type: array
      items:
        $ref: '#/components/schemas/Order'
      example: [
        {
          "id": "1",
          "instrument": "EURUSD",
          "qty": 101,
          "side": "buy",
          "type": "limit",
          "avgPrice": 0,
          "limitPrice": 1.2093,
          "duration": {
            "type": "gtt",
            "datetime": 1548406235
          },
          "status": "working",
          "customFields": [{
            "id": "commission",
            "value": "1.25"
          }]
        }
      ]

    IndividualPositionMessage:
      title: IndividualPositionMessage
      type: array
      items:
        $ref: '#/components/schemas/IndividualPosition'
      example: [
        {
            "id": "ip1",
            "instrument": "EUR/USD",
            "qty": 50,
            "side": "buy",
            "price": 1.13470,
            "pl": 555.44
        },
        {
            "id": "ip2",
            "instrument": "EUR/USD",
            "qty": 40,
            "side": "buy",
            "price": 1.12,
            "pl": 405.44
        }
      ]

    NetPositionMessage:
      title: NetPositionMessage
      type: array
      items:
        $ref: '#/components/schemas/NetPosition'
      example: [
        {
            "id": "np1",
            "instrument": "EUR/USD",
            "qty": 90,
            "side": "buy",
            "longQty": 90,
            "shortQty": 0,
            "avgPrice": 1.12470
        }
      ]

    OrderStatus:
      title: OrderStatus
      type: object
      properties:
        status:
          description: >
            String constants to describe an order status.


            `Status`  | Description

            ----------|-------------

            placing   | order is not created on a broker's side yet

            inactive  | bracket order is created but waiting for a base order to
            be filled

            working   | order is created but not fully executed yet

            rejected  | order is rejected for some reason

            filled    | order is fully executed

            cancelled  | order is cancelled
          type: string
          enum:
            - placing
            - inactive
            - working
            - rejected
            - filled
            - cancelled
      required:
        - status
      example: 'working'

    PermissionGroups:
      title: PermissionGroups
      type: object
      required:
        - groups
      properties:
        groups:
          description: Groups list. Each element of this array is a group identifier.
          type: array
          items:
            type: string
      additionalProperties: false
      example: { groups: [ 'broker_futures', 'broker_stocks', 'broker_forex' ]}

    PermissionsResponse:
      title: PermissionsResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/PermissionGroups'
      required:
        - s
        - d
      additionalProperties: false

    PlaceModifyPreviewOrderCommon:
      title: PlaceModifyPreviewOrderCommon
      type: object
      properties:
        qty:
          description: The number of units.
          type: number
          example: 1
        limitPrice:
          description: Limit Price for Limit or StopLimit order.
          type: number
          example: 1.23456
        stopPrice:
          description: Stop Price for Stop or StopLimit order.
          type: number
        durationType:
          description: Duration ID (if supported).
          type: string
          example: gtt
        durationDateTime:
          description: Expiration datetime Unix timestamp (if supported by duration type).
          type: number
          example: 1548406235
        stopLoss:
          description: StopLoss price (if supported).
          type: number
        trailingStopPips:
          description: Distance from the stop loss level to the current market price in pips (if supported by the broker).
          type: number
        takeProfit:
          description: TakeProfit price (if supported).
          type: number
        digitalSignature:
          description: Digital signature (if supported).
          type: string
          example: 954345868
        currentAsk:
          description: Current ask price for the instrument that the user sees in the order panel.
          type: number
          example: 1.25
        currentBid:
          description: Current bid price for the instrument that the user sees in the order panel.
          type: number
          example: 1.23
      additionalProperties: false

    PlacePreviewOrderCommon:
      title: PlacePreviewOrderCommon
      type: object
      properties:
        instrument:
          description: Instrument.
          type: string
          example: EURUSD
        side:
          description: Side.
          type: string
          enum:
            - buy
            - sell
          example: buy
        type:
          description: Type.
          type: string
          enum:
            - market
            - stop
            - limit
            - stoplimit
          example: limit
      additionalProperties: false

    Position:
      title: Position
      type: object
      properties:
        id:
          description: Unique identifier.
          type: string
          example: '1'
        instrument:
          description: Instrument name that is used on a broker's side.
          type: string
          example: 'EURUSD'
        qty:
          description: Quantity.
          type: number
          example: 1
        side:
          description: Side.
          type: string
          enum:
            - buy
            - sell
          example: 'buy'
        avgPrice:
          description: Average price of position trades.
          type: number
          example: 1.1347091
        unrealizedPl:
          description: Unrealized (open) profit/loss.
          type: number
          example: 19.4739
        message:
          $ref: '#/components/schemas/Message'
        customFields:
          type: array
          description: |
            Localized position custom fields values data.
            Custom fields are configured in the [/config](#operation/getConfiguration) endpoint response.
          items:
            $ref: '#/components/schemas/CustomFieldsValueItem'
          minItems: 1
      required:
        - id
        - instrument
        - qty
        - side
        - avgPrice
      additionalProperties: false

    NetPosition:
      title: NetPosition
      type: object
      properties:
        id:
          description: Unique identifier.
          type: string
          example: '1'
        instrument:
          description: Instrument name that is used on a broker's side.
          type: string
          example: 'EURUSD'
        qty:
          description: Net quantity.
          type: number
          example: 1
        longQty:
          description: Net quantity of long positions.
          type: number
          example: 2
        shortQty:
          description: Net quantity of short positions.
          type: number
          example: 1
        avgPrice:
          description: Average price of net position trades.
          type: number
          example: 1.23
        customFields:
          type: array
          description: |
            Localized net position custom fields values data.
            Custom fields are configured in the [/config](#operation/getConfiguration) endpoint response.
          items:
            $ref: '#/components/schemas/CustomFieldsValueItem'
          minItems: 1
        message:
          $ref: '#/components/schemas/Message'
      required:
        - id
        - instrument
        - qty

    IndividualPosition:
      title: IndividualPosition
      type: object
      properties:
        id:
          description: Unique identifier.
          type: string
          example: '1'
        instrument:
          description: Instrument name that is used on a broker's side.
          type: string
          example: EURUSD
        date:
          description: The date when the individual position was opened. UNIX timestamp in milliseconds.
          type: string
          example: '1683818630353'
        qty:
          description: Quantity.
          type: number
          example: 1
        side:
          description: Side.
          type: string
          enum:
            - buy
            - sell
          example: 'buy'
        price:
          description: Individual position's price.
          type: number
          example: 23.274
        unrealizedPl:
          description: Individual position's profit/loss.
          type: number
          example: 3.094
        customFields:
          type: array
          description: |
            Localized individual position custom fields values data.
            Custom fields are configured in the [/config](#operation/getConfiguration) endpoint response.
          items:
            $ref: '#/components/schemas/CustomFieldsValueItem'
          minItems: 1
        message:
          $ref: '#/components/schemas/Message'
      required:
        - id
        - instrument
        - qty
        - side
        - price
        - date

    DeletePositionRequestBody:
      title: DeletePositionRequestBody
      type: object
      properties:
        amount:
          type: number
          description: Amount to close. This property is used if supportPartialClosePosition flag is `true`.
      example: {
        "amount": 5,
      }

    DeleteIndividualPositionRequestBody:
      title: DeleteIndividualPositionRequestBody
      type: object
      properties:
        amount:
          type: number
          description: Amount to close. This property is used if supportPartialCloseIndividualPosition flag is `true`.
      example: {
        "amount": 5,
      }

    PositionCustomFields:
      title: PositionCustomFields
      type: array
      description: |
        Additional position columns to be displayed in the positions table of the Account manager.
        The positions table has a default set of columns that can be extended using this configuration.
        The `supportPositionCustomFields` flag should be enabled in the [account configuration](https://www.tradingview.com/rest-api-spec/#operation/getAccounts).
        Data of the additional fields is filled from the
        [/positions](https://www.tradingview.com/rest-api-spec/#operation/getPositions) `customFields` value.
      items:
        $ref: '#/components/schemas/AccountManagerColumn'
      minItems: 1
      example: [
        {
          "id": "exchangeFee",
          "title": "Exchange fee",
          "tooltip": "Exchange fee for the position",
          "alignment": "right"
        },
        {
          "id": "routeFee",
          "title": "Route fee",
          "tooltip": "Route fee for the position",
          "alignment": "right"
        }
      ]

    PositionsResponse:
      title: PositionsResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/Position'
      required:
        - s
        - d
      additionalProperties: false

    PositionsMessage:
      title: PositionsMessage
      type: array
      items:
        $ref: '#/components/schemas/Position'

    PingMessage:
      title: PingMessage
      type: object
      description:
        A ping message to ensure that the connection persists. Must be sent every 5 seconds if there were no entity updates.
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
      required:
        - s

    ErrorMessage:
      title: ErrorMessage
      $ref: '#/components/schemas/Error'
      additionalProperties: false

    Error:
      type: object
      properties:
        s:
          type: string
          description: Status will always be `error`.
          enum:
            - error
          example: 'error'
        errmsg:
          type: string
          description: Error message.
          example: 'An error occurred.'
      required:
        - s
        - errmsg

    NetPositionsResponse:
      title: NetPositionsResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/NetPosition'
      required:
        - s
        - d
      additionalProperties: false

    IndividualPositionsResponse:
      title: IndividualPositionsResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          type: array
          items:
            $ref: '#/components/schemas/IndividualPosition'
      required:
        - s
        - d
      additionalProperties: false

    ModifyPositionCommonRequestBody:
      type: object
      properties:
        stopLoss:
          description: StopLoss price.
          type: number
          example: 1.283568
        trailingStopPips:
          description: Distance from the stop loss level to the order price in pips.
          type: number
        takeProfit:
          description: TakeProfit price.
          type: number
          example: 1.294436
        currentAsk:
          description: Current ask price for the instrument that the user sees in the order panel.
          type: number
          example: 1.287367
        currentBid:
          description: Current bid price for the instrument that the user sees in the order panel.
          type: number
          example: 1.28554

    ModifyPositionRequestBody:
      title: ModifyPositionRequestBody
      allOf:
        - $ref: '#/components/schemas/ModifyPositionCommonRequestBody'
        - type: object
          properties:
            side:
              description: |
                New side of the position. This parameter is used to reverse the position,
                if the `supportNativeReversePosition` flag is enabled in the account config.
                Please see the [/accounts](#operation/getAccounts) endpoint.
              type: string
              enum:
                - 'buy'
                - 'sell'

    ModifyIndividualPositionRequestBody:
      title: ModifyIndividualPositionRequestBody
      allOf:
        - $ref: '#/components/schemas/ModifyPositionCommonRequestBody'

    PreviewLeverage:
      title: PreviewLeverage
      type: object
      properties:
        infos:
          description: An array of information messages if any.
          type: array
          items:
            type: string
          minItems: 1
        warnings:
          description: An array of warning messages if any.
          type: array
          items:
            type: string
          minItems: 1
        errors:
          description: An array of error messages if any.
          type: array
          items:
            type: string
          minItems: 1
      additionalProperties: false
      example: {
        infos: ['Borrowing limit at current leverage 0.3057 BTC or 0.00000 USDT', 'Required position margin at current leverage 0.3057 BTC'],
        warnings: ['Your leverage is comparatively high. Please be aware of risks'],
        errors: ['Invalid leverage value'],
      }

    PreviewLeverageResponse:
      title: PreviewLeverageResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/PreviewLeverage'
      required:
        - s
        - d
      additionalProperties: false

    PostOrderResponse:
      title: PostOrderResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          allOf:
            - type: object
              properties:
                orderId:
                  type: string
                  description: New order identifier.
                  example: '1'
            - $ref: '#/components/schemas/TransactionId'
          additionalProperties: false
      required:
        - s
        - d
      additionalProperties: false

    PreviewOrder:
      title: PreviewOrder
      type: object
      properties:
        confirmId:
          description: An optional identifier of an order. It is sent back to the server in the place order request.
          type: string
          example: '112358'
        sections:
          type: array
          items:
            $ref: '#/components/schemas/OrderPreviewSection'
        warnings:
          description: Optional array of text warnings.
          type: array
          items:
            type: string
        errors:
          description: Optional array of text errors.
          type: array
          items:
            type: string
      required:
        - sections
      additionalProperties: false
      example: {
        confirmId: 112358,
        sections: [
          {
            header: Estimated,
            rows: [
              {
                title: Estimated Commission,
                value: 0.01
              },
              {
                title: Estimated Price,
                value: 4091
              }
            ]
          },
          {
            rows: [
              {
                title: Margin Value,
                value: 12100
              },
              {
                title: Time In Force,
                value: DAY
              }
            ]
          }
        ],
        warnings: [
            Estimated money impact is for main order only.,
            Order can be executed outside regular trading hours.
        ],
        errors: [
            Failed to build order confirmation.
        ]
      }

    PreviewOrderResponse:
      title: PreviewOrderResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/PreviewOrder'
      required:
        - s
        - d
      additionalProperties: false

    PullingInterval:
      title: PullingInterval
      type: object
      description: |
        Time intervals in milliseconds to pull data from the server.
        Please note: if the pulling interval is set above the maximum value then the maximum value will be used explicitly.
      properties:
        quotes:
          description: Time interval in milliseconds to request quote and level 2 depth updates.
          type: number
          default: 500
          maximum: 1000
          example: 500
        orders:
          description: Time interval in milliseconds to request orders.
          type: number
          default: 500
          maximum: 1000
          example: 500
        positions:
          description: Time interval in milliseconds to request positions.
          type: number
          default: 1000
          maximum: 1500
          example: 1000
        accountManager:
          description: Time interval in milliseconds to update Account manager tables.
          type: number
          default: 500
          maximum: 1500
          example: 1000
        balances:
          description: Time interval in milliseconds to request balances.
          type: number
          default: 1000
          maximum: 1500
          example: 1000
      additionalProperties: false

    QuotesResponse:
      title: QuotesResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/Quotes'
      required:
        - s
        - d
      additionalProperties: false

    QuotesMessage:
      title: QuotesMessage
      $ref: '#/components/schemas/Quotes'

    Quotes:
      title: Quotes
      type: array
      items:
        type: object
        description: Price response for an instrument
        properties:
          s:
            $ref: '#/components/schemas/Status'
          n:
            description: Symbol name. Should be equal to the requested one.
            type: string
            example: 'EURUSD'
          v:
            $ref: '#/components/schemas/SingleQuote'
        required:
          - n
          - v
        additionalProperties: false
      minItems: 1

    SetLeverage:
      title: SetLeverage
      type: object
      properties:
        leverage:
          description: Leverage value confirmed by broker.
          type: number
      required:
        - leverage
      additionalProperties: false
      example: {
        "leverage": 25,
      }

    SetLeverageResponse:
      title: SetLeverageResponse
      type: object
      properties:
        s:
          $ref: '#/components/schemas/OkStatus'
        d:
          $ref: '#/components/schemas/SetLeverage'
      required:
        - s
        - d

    SingleField:
      title: SingleField
      type: string
      enum:
        - brokerSymbol
      description: Constant. Set it to `brokerSymbol`.
      example: brokerSymbol

    SingleMapping:
      title: SingleMapping
      type: object
      description: Map of Broker instrument name to TradingView instrument name.
      properties:
        f:
          description: An array with the only one string element &ndash; broker symbol name.
          type: array
          items:
            type: string
            example: 'EURUSD'
          minItems: 1
        s:
          description: TradingView symbol name with a prefix (AA:XXXX).
          type: string
          example: 'FX_IDC:EURUSD'
      required:
        - f
        - s
      additionalProperties: false

    SingleQuote:
      title: SingleQuote
      type: object
      description: Price and restriction data for an instrument.
      properties:
        ask:
          description: Ask price.
          type: number
          example: 1.13836
        bid:
          description: Bid price.
          type: number
          example: 1.13834
        buyPipValue:
          description: Value of 1 pip in the account currency, used for calculating risks and trade value in the Order dialog for buy orders.
          type: number
          example: 1
        sellPipValue:
          description: Value of 1 pip in the account currency, used for calculating risks and trade value in the Order dialog for sell orders.
          type: number
          example: 1
        hardToBorrow:
          description: Specify if the instrument is hard to borrow.
          type: boolean
          default: false
        notShortable:
          description: Specify if the instrument is not shortable.
          type: boolean
          default: false
        halted:
          description: Specify if the instrument is halted.
          type: boolean
          default: false
      required:
        - ask
        - bid
      additionalProperties: false

    Status:
      title: Status
      type: string
      enum:
        - ok
        - error

    StopLossPercentValidationRule:
      title: StopLossPercentValidationRule
      description: Stop Loss Percent Price Range Rule applied to field validation in the Order dialog.
      type: object
      properties:
        id:
          type: string
          enum:
            - slPercent
        options:
          type: object
          properties:
            min:
              description: Minimal value in percentage of the stop loss price relative to the parent order price.
              type: number
              example: 10
            max:
              description: Maximum value in percentage of the stop loss price relative to the parent order price.
              type: number
              example: 90
          required:
            - min
            - max
          additionalProperties: false
      required:
        - id
        - options
      additionalProperties: false

    StopPercentValidationRule:
      title: StopPercentValidationRule
      description: Stop Percent Price Range Rule applied to the order price field validation in the Order dialog.
      type: object
      properties:
        id:
          type: string
          enum:
            - stopPercent
        options:
          type: object
          properties:
            min:
              description: |
                Minimal value in percentage of the stop price relative to the current price.
              type: number
              example: 10
            max:
              description: |
                Maximum value in percentage of the stop price relative to the current price.
              type: number
              example: 20
          required:
            - min
            - max
          additionalProperties: false
      required:
        - id
        - options
      additionalProperties: false

    StreamingAskBidTradeItem:
      title: StreamingAskBidTradeItem
      type: object
      description: Ask or Bid.
      properties:
        id:
          description: Symbol
          type: string
          example: 'EURUSD'
        p:
          description: Price.
          type: number
          example: 1.47245
        s:
          description: Size.
          type: number
          example: 100
        t:
          description: Unix time UTC.
          type: number
          example: 1547942400
      required:
        - id
        - p
        - t
        - s

    StreamingAskItemType:
      title: StreamingAskItemType
      type: object
      description: Ask
      properties:
        f:
          description: Should always be `a`.
          type: string
          example: 'a'

    StreamingAskResponse(deprecated):
      title: StreamingAskResponse(deprecated)
      allOf:
        - $ref: '#/components/schemas/StreamingAskItemType'
        - $ref: '#/components/schemas/StreamingAskBidTradeItem'

    StreamingBidItemType:
      title: StreamingBidItemType
      type: object
      description: Bid
      properties:
        f:
          description: Should always be `b`.
          type: string
          example: 'b'

    StreamingBidResponse(deprecated):
      title: StreamingBidResponse(deprecated)
      allOf:
        - $ref: '#/components/schemas/StreamingBidItemType'
        - $ref: '#/components/schemas/StreamingAskBidTradeItem'

    StreamingDailyBarResponse:
      title: StreamingDailyBarResponse
      allOf:
        - $ref: '#/components/schemas/StreamingDailyBarType'
        - type: object
          description: Daily bar
          properties:
            id:
              description: Symbol
              type: string
              example: 'EURUSD'
            o:
              description: Open.
              type: number
              example: 3667
            h:
              description: High.
              type: number
              example: 3667.24
            l:
              description: Low.
              type: number
              example: 3661.55
            c:
              description: Close.
              type: number
              example: 3662.25
            v:
              description: Volume.
              type: number
              example: 34
            t:
              description: Unix timestamp in seconds. Timestamp time should be always 00:00 at the start of the day.
              type: number
              example: 1547931600
          required:
            - id
            - o
            - h
            - l
            - c
            - v
            - t

    StreamingDailyBarType:
      title: StreamingDailyBarType
      type: object
      description: Daily Bar
      properties:
        f:
          description: Should always be `d`.
          type: string
          example: 'd'
      required:
        - f

    StreamingHeartbeatResponse:
      title: StreamingHeartbeatResponse
      type: object
      properties:
        heartbeat:
          type: integer
          description: Current Unix timestamp in seconds.
      example: { "heartbeat": 1609255644 }

    StreamingQuoteResponse:
      title: StreamingQuoteResponse
      type: object
      properties:
        f:
          type: string
          description: Should always be `q`.
        id:
          type: string
          description: Symbol.
        t:
          type: number
          description: Unix time UTC.
        ap:
          type: number
          description: Best ask price.
        as:
          type: number
          description: Best ask size.
        bp:
          type: number
          description: Best bid price.
        bs:
          type: number
          description: Best bid size.
      required:
        - f
        - id
        - t
        - ap
        - as
        - bp
        - bs
      example:  { "f": "q", "id": "EURUSD", "t": 1547942400, "ap": 1.47245, "as": 100, "bp": 1.47245, "bs": 100 }

    StreamingTradeItemType:
      title: StreamingTradeItemType
      type: object
      description: Trade
      properties:
        f:
          description: Should always be `t`.
          type: string
          example: 't'
      required:
        - f

    StreamingTradeResponse:
      title: StreamingTradeResponse
      allOf:
        - $ref: '#/components/schemas/StreamingTradeItemType'
        - $ref: '#/components/schemas/StreamingAskBidTradeItem'

    SuccessResponse:
      title: SuccessResponse
      type: object
      properties:
        s:
          type: string
          description: Status will always be `ok`.
          enum:
            - ok
          example: 'ok'
      required:
        - s
      additionalProperties: false

    SuccessResponseWithTransactionId:
      title: SuccessResponseWithTransactionId
      allOf:
        - $ref: '#/components/schemas/SuccessResponse'
        - type: object
          properties:
            d:
              $ref: '#/components/schemas/TransactionId'

    SymbolInfoArrays:
      title: SymbolInfoArrays
      type: object
      description: |
        SymbolInfo is an object containing symbols metadata. Each value of this  object is an array of values which
        size is equal to symbols count or a single value that is applied to all symbols.
        You can use a single value for all fields except for `supported-resolutions` and `intraday-multipliers`.
      properties:
        symbol:
          description: |
            It is a symbol name which is a string that users see. Also, the symbol name is used for data requests
            if you are not using tickers. `symbol` must contain uppercase letters and can optionally contain numbers,
            dots, or underscores. Learn more about `symbol`
            in the [manual](https://www.tradingview.com/broker-api-docs/data/Symbol_info.html#symbol-naming-rules).
          type: array
          items:
            type: string
          minItems: 1
          example: ['VIXG2019', 'AAPLE', 'EURUSD']
        description:
          description: |
            Description of a symbol. Will be displayed in the chart legend for this symbol.
          type: array
          items:
            type: string
          example: ['Volatility Index','Apple Inc','EUR/USD']
        currency:
          description: |
            Symbol currency, also named as counter currency. If a symbol is a currency pair, then the currency field has
            to contain the second currency of this pair. For example, `USD` is a currency for `EURUSD` ticker.
            Fiat currency must meet the ISO 4217 standard.
          type: array
          items:
            type: string
            nullable: true
          nullable: true
          default: null
          example: ['USD', 'USD', 'USD']
        base-currency:
          description: |
            For currency pairs only. This field contains the first currency of the pair. For example, base currency for
            `EURUSD` ticker is `EUR`. Fiat currency must meet the ISO 4217 standard.
          type: array
          items:
            type: string
            nullable: true
          nullable: true
          default: null
          example: [null, null, 'EUR']
        exchange-listed:
          description: Short name of exchange where this symbol is listed.
          type: array
          items:
            type: string
          example: ['CBOE', 'NASDAQ', 'FOREX']
        exchange-traded:
          description: Short name of exchange where this symbol is traded.
          type: array
          items:
            type: string
          example: ['CBOE', 'NASDAQ', 'FOREX']
        minmovement:
          description: Minimal integer price change.
          type: array
          items:
            type: number
          example: [1.0, 1.0, 1.0]
        minmovement2:
          description: This is a number for complex price formatting cases. The default value is `0`.
          type: array
          items:
            type: number
          example: [0, 0, 0]
        fractional:
          description: |
            Boolean showing whether this symbol wants to have complex price
            formatting (see `minmovement2`) or not. The default value is `false`.
          type: array
          items:
            type: boolean
          example: [false, false, false]
        pricescale:
          description: |
            Indicates how many decimal points the price has. For example, if the price has 2 decimal points (ex., 300.01),
            then `pricescale` is `100`. If it has 3 decimals, then `pricescale` is `1000` etc. If the price doesn't have decimals,
            set `pricescale` to `1`.
          type: array
          items:
            type: number
          example: [100, 100, 100000]
        root:
          description: |
            Root of the features. It's required for futures symbol types only.
            Provide a null value for other symbol types.
          type: array
          items:
            type: string
            nullable: true
          nullable: true
          default: null
          example: ['VIX', null, null]
        root-description:
          description: |
            Short description of the futures root that will be displayed in the symbol search.
            It's required for futures only. Provide a null value for other symbol types.
          type: array
          items:
            type: string
            nullable: true
          nullable: true
          default: null
          example: ['Volatility Index', null, null]
        has-intraday:
          description: |
            Boolean value showing whether the symbol includes intraday (minutes)
            historical data. If it's `false` then all buttons for intraday resolutions
            will be disabled for this particular symbol. If it is set to `true`, all
            resolutions that are supplied directly by the datafeed must be provided
            in `intraday-multipliers` array. The default value is `true`.
          type: array
          items:
            type: boolean
          example: [true, true, true]
        has-no-volume:
          description: Boolean showing whether the symbol includes volume data or not. The default value is `false`.
          type: array
          items:
            type: boolean
          example: [false, false, true]
        type:
          description: Symbol type (forex/stock etc.).
          type: array
          items:
            $ref: '#/components/schemas/SymbolType'
          example: ['futures', 'stock', 'forex']
        is-cfd:
          description: Boolean value showing whether the symbol is CFD. The base instrument type is set using the type field.
          type: array
          items:
            type: boolean
        ticker:
          description: |
            This is a unique identifier for this particular symbol in your symbology.
            If you specify this property then its value will be used for all data requests for this symbol.
          type: array
          items:
            type: string
          example: ['VIXG2019', 'AAPLE', 'EURUSD']
        timezone:
          description: |
            Timezone of the exchange for this symbol. We expect to get the name of the time zone in olsondb format.
          type: array
          items:
            type: string
            enum:
              - America/New_York
              - America/Los_Angeles
              - America/Chicago
              - America/Phoenix
              - America/Toronto
              - America/Vancouver
              - America/Argentina/Buenos_Aires
              - America/El_Salvador
              - America/Sao_Paulo
              - America/Bogota
              - Europe/Moscow
              - Europe/Athens
              - Europe/Berlin
              - Europe/London
              - Europe/Madrid
              - Europe/Paris
              - Europe/Warsaw
              - Australia/Sydney
              - Australia/Brisbane
              - Australia/Adelaide
              - Australia/ACT
              - Asia/Almaty
              - Asia/Ashkhabad
              - Asia/Tokyo
              - Asia/Taipei
              - Asia/Singapore
              - Asia/Shanghai
              - Asia/Seoul
              - Asia/Tehran
              - Asia/Dubai
              - Asia/Kolkata
              - Asia/Hong_Kong
              - Asia/Bangkok
              - Pacific/Auckland
              - Pacific/Chatham
              - Pacific/Fakaofo
              - Pacific/Honolulu
              - America/Mexico_City
              - Africa/Johannesburg
              - Asia/Kathmandu
              - US/Mountain
              - Etc/UTC
          example: ['America/New_York', 'America/New_York', 'America/New_York']
        session-regular:
          description: |
            Session time format is HHMM-HHMM. E.g., a session that starts at 9:30 am and ends at 4:00 pm should look like `0930-1600`.
            There is a special case for symbols traded 24/7 (e.g., Bitcoin and other cryptocurrencies): the session string should be `24x7`.
            To specify an overnight session set start time greater than end time (ie, `1700-0900`).
            Session time is expected to be in exchange time zone.
          type: array
          items:
            type: string
          example: ['0000-2359:23456', '0930-1600', '1700-1700']
        session-extended:
          description: |
            An extended session, includes `session-premarket` and `session-postmarket`.
            The start time should be earlier or be equal to the start time of the `session-regular`
            and be equal to the start time of the `session-premarket` if it exists.
          type: array
          items:
            type: string
            nullable: true
          example: ['0000-2359:23456', '0400-2000', '1700-1700']
        session-premarket:
          description: |
            An additional session before `session-regular`. The start time should be equal to the start time of the `session-extended`.
            The end time should be equal or less than the start time of the `session-regular`.
          type: array
          items:
            type: string
            nullable: true
          example: [null, '0400-0930', null]
        session-postmarket:
          description: |
            An additional session after the `session-regular`. The start time should be equal or greater
            than the end time of the `session-regular`. The end time should be equal to the end time of the `session-extended`.
          type: array
          items:
            type: string
            nullable: true
          example: [null, '1600-2000', null]
        supported-resolutions:
          description: |
            An array of resolutions which should be enabled in resolutions picker
            for this symbol. Each item of an array is expected to be a string.
          type: array
          items:
            type: array
            items:
              type: string
          default: []
          example: [['1', '3', '5', '15', '30', '60', '240', 'D', 'W'], ['1', '3', '5', '15', '30', '60', '240', 'D', 'W'], ['1', '3', '5', '15', '30', '60', '240', 'D', 'W']]
        has-daily:
          description: |
            The boolean value showing whether data feed has its own daily
            resolution bars or not. If `has-daily` = `false` then Charting Library
            will build the respective resolutions using 1-minute bars by itself.
            If not, then it will request those bars from the data feed.
            The default value is `true`.
          type: array
          items:
            type: boolean
          example: [true, true, true]
        intraday-multipliers:
          description: |
            This is an array containing intraday resolutions (in minutes) that the data feed may provide.
            E.g., if the data feed supports resolutions such as `["1", "5", "15"]`, but has 1-minute bars
            for some symbols then you should set `intraday-multipliers` of this symbol to `[1]`. This will
            make Charting Library build 5 and 15-minute resolutions by itself.
          type: array
          items:
            type: array
            items:
              type: string
          default: []
          example: [['1', '3', '5', '15', '30', '60', '240'], ['1', '3', '5', '15', '30', '60', '240'], ['1', '3', '5', '15', '30', '60', '240']]
        has-weekly-and-monthly:
          description: |
            The boolean value showing whether data feed has its own weekly
            and monthly resolution bars or not. If `has-weekly-and-monthly` = `false`
            then Charting Library will build the respective resolutions using daily
            bars by itself. If not, then it will request those bars from the data feed.
            The default value is `false`.
          type: array
          items:
            type: boolean
          example: [false, false, false]
        pointvalue:
          description: |
            The currency value of a single whole unit price change in the instrument's currency.
            If the value is not provided it is assumed to be `1`.
          type: array
          items:
            type: number
          example: [10, 1, 0.00001]
        expiration:
          description: |
            Expiration of the futures in the following format: YYYYMMDD. Required for futures type symbols only.
          type: array
          items:
            type: number
            nullable: true
          nullable: true
          default: null
          example: [20190213, null, null]
        bar-source:
          description: The principle of building bars. The default value is `trade`.
          type: array
          items:
            type: string
            enum:
              - bid
              - ask
              - mid
              - trade
          example: ['trade', 'bid', 'ask']
        bar-transform:
          description: The principle of bar alignment. The default value is `none`.
          type: array
          items:
            type: string
            enum:
              - none
              - openprev
              - prevopen
          example: ['openprev', 'openprev', 'none']
        bar-fillgaps:
          description: |
            Is used to create the zero-volume bars in the absence of any trades
            (i.e. bars with zero volume and equal OHLC values ).
            The default value is `false`.
          type: array
          items:
            type: boolean
          example: ['true', 'true', 'false']
        isin:
          description:
            International Securities Identification Number. It's needed for classic stock instruments only.
          type: array
          items:
            type: string
          example: ['ZAE000190252', 'ZAE0001201977', 'ZAE000567432']
        wkn:
          description:
            German Securities Identification Code. It's needed for classic German stock instruments only.
          type: array
          items:
            type: string
          example: ['A12ABB', 'A66RTT', 'A13DDE']
      required:
        - symbol
        - description
        - currency
        - exchange-listed
        - exchange-traded
        - minmovement
        - pricescale
        - timezone
        - type
        - session-regular

    SymbolInfoResponse:
      title: SymbolInfoResponse
      allOf:
        - $ref: '#/components/schemas/SuccessResponse'
        - $ref: '#/components/schemas/SymbolInfoArrays'

    SymbolMapping:
      title: SymbolMapping
      type: object
      description: Map of Broker instrument names and TradingView instrument names.
      properties:
        symbols:
          type: array
          items:
            $ref: '#/components/schemas/SingleMapping'
        fields:
          description: 'Array with the only one element `[''brokerSymbol'']`.'
          type: array
          items:
            $ref: '#/components/schemas/SingleField'
          minItems: 1
          maxItems: 1
      additionalProperties: false
      example:
        {
          "symbols": [
            {
              "f": ["EURUSD"],
              "s": "FX_IDC:EURUSD"
            },
            {
              "f": ["AAPLE"],
              "s": "NASDAQ:AAPLE"
            },
          ],
          "fields": ["brokerSymbol"]
        }

    SymbolType:
      title: SymbolType
      type: string
      description: Symbol type (forex, stock, etc.).
      enum:
        - stock
        - dr
        - right
        - bond
        - warrant
        - structured
        - index
        - forex
        - futures
        - crypto
        - cfd
      example: forex

    TakeProfitPercentValidationRule:
      title: TakeProfitPercentValidationRule
      description: Take Profit Percent Price Range Rule applied to field validation in the Order dialog.
      type: object
      properties:
        id:
          type: string
          enum:
            - tpPercent
        options:
          type: object
          properties:
            min:
              description: Minimal value in percentage of the take profit price relative to the parent order price.
              type: number
              example: 10
            max:
              description: Maximum value in percentage of the take profit price relative to the parent order price.
              type: number
              example: 90
          required:
              - min
              - max
          additionalProperties: false
      required:
        - id
        - options
      additionalProperties: false

    TransactionId:
      title: TransactionId
      description: |
        Transaction id is used in these endpoints as an identifier for latency tracking using [/trackLatency](#operation/trackLatency):
          [/placeOrder](#operation/placeOrder),
          [/modifyOrder](operation/#operation/modifyOrder),
          [/cancelOrder](#operation/cancelOrder),
          [/modifyPosition](#operation/modifyPosition),
          [/closePosition](#operation/closePosition)
        This field is used only if `supportTrackLatency` flag is set to true.
      type: object
      properties:
        transactionId:
          description: An unique transaction identifier
          type: number
