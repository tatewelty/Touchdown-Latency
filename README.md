# Touchdown Latency  

### Idea:  
When a touchdown is scored, there is a time delay between when the fans at the stadium celebrate, when the score online updates, and when your fantasy football players get their fantasy points.  With the introduction of prediction markets like Kalshi and Polymarket, which offer markets on specific players scoring a touchdown, there is theoretically a time-based edge.  If you can place a trade on a specific player that just scored before the market updates, you can profit from the resting orders saying that player will not score a touchdown.  
  
To test this idea, I used the ESPN API to detect when touchdowns are scored, and then placed an order on Kalshi that the specified player would score a touchdown.  

### Outcome:  
#### Programmatic Data
Early in the season, using the ESPN API to Kalshi pipeline was sometimes profitable.  The edge was small and inconsistent because I was never the first to execute trades with this information.  As institutional algorithms gained confidence, the market adjusted quickly, and prices rose to 99 cents to win 1 cent, effectively eliminating the edge.  

| Pipeline | Description |
|-------|------|
|**My Pipeline**| NFL event -> Genius Sports (NFL Data provider) -> intended delay -> ESPN API -> I pulled this data -> I placed a trade |
|**Institutional Pipeline**| NFL event -> Genius Sports (NFL Data provider)-> Immediately to Institution -> trade placed |


Institutional traders pay tens of thousands per month for direct feed access and skip many intermediate steps.  Their latency is much lower, making it impossible for an individual to compete.  Even institutions have limited profits, since only the fastest order makes a profit and there are multiple institutions fighting for that profit.  Publicly available data sources do not provide a reliable edge.

#### On-Site Data  
If programmatic data is too slow, the only alternative is to physically go to the game and try to beat the data feed in that way, since there is no faster data feed than seeing it in real time.  However, sportsbooks have already thought of this and have people physically at the game whose entire job is to input plays quickly as a parallel data stream to the official data feed.  Human error is introduced here, so it is possible an individual can gain a time advantage if they are quick with the input, and the sportsbook's person is slow.  

#### Profitability Based On Risk Tolerance  
Even in the perfect scenario where a touchdown is scored, you act quickly while the sportsbook person operates slowly, risk still remains.  You may submit your trade faster than everyone else, but a flag or overturned call could nullify the touchdown, leaving you with a large position on a player who may never score.  

Sportsbooks handle these uncertainties differently, adjusting their orders based on confidence the touchdown stands and the likelihood that the player scores a touchdown later in the game.  These decisions are made and traded on in milliseconds, leaving no time for an individual to analyze the situation and make an educated decision on the price and quantity of their trade.

### My Code And Methodology:  
The code is in `Touchdown_Latency.ipynb`, an interactive notebook you can experiement with.  
Required packages are in `requirements.txt`, and can be installed via terminal with: `pip install -r requirements.txt`  

#### Pulling Data From ESPN API  
1.  Use `get_live_games` to retrieve live NFL games.  ESPN uses a unique identifier for each NFL game.
2.  From the game id, call `get_plays_from_game` to fetch all plays that have occurred.
3.  Parse through those plays to find touchdowns and identify who scored the touchdown with `get_scorer_from_play`
```python
live_games = get_live_games()  
for game_id in live_games:  
  plays = get_plays_from_game(game_id)  
  for play in plays:  
    if play.get("scoringPlay"):  
      scorer = get_scorer_from_play(play)  
```
  
#### Kalshi Data and Orders  
1. On Kalshi find the specific market id for the NFL game.  Example: `kxnflgame-25oct27waskc`  
2. Pass the game id to `get_kalshi_game_information` to get all available markets, including anytime touchdown markets
3. Identify the market for the specific player scoring the touchdown.  Example: `KXNFLANYTD-25OCT27WASKC-KCRRICE4`
4. Select the price and quantity for your order
5. Post your order to Kalshi with `post`

```python
Example_Kalshi_Game = 'KXNFLANYTD-25OCT27WASKC'
Example_Kalshi_Game_Scorer = 'Rice'
Example_Kalshi_Game_Information = get_kalshi_game_information(Example_Kalshi_Game)
Example_Scorer_Market = next((m["ticker"]for m in Example_Kalshi_Game_Information['markets']if Example_Kalshi_Game_Scorer.lower() in m.get("no_sub_title", "").lower()),None)
order_data = {"ticker": Example_Scorer_Market,"action": "buy","side": "yes","count": purchase_quantity,"type": "limit","yes_price": purchase_price}
post(private_key, access_key, add_url, order_data, base_url=BASE_URL)
```
Successful order example:  
![order_placement](https://github.com/user-attachments/assets/0dece1eb-7b9b-42e9-8613-798dd7021d9e)  
