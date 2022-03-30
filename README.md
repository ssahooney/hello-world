import time
from pybithumb import Bithumb
import pybithumb
import datetime

con_key = ""
sec_key = ""

bithumb = pybithumb.Bithumb(con_key, sec_key)

#매수주문
def buy_crypto_currency(ticker):
    krw = bithumb.get_balance(ticker)[2]
    sell_price = bithumb.get_current_price(ticker)
    unit = krw/float(sell_price) * 0.995
    return bithumb.buy_market_order(ticker, unit)

#매도주문
def sell_crypto_currency(ticker):
    unit = bithumb.get_balance(ticker)[0]
    return bithumb.sell_market_order(ticker, unit)

#현재상태
def get_status(ticker) :
    krw = bithumb.get_balance(ticker)[2]
    if krw > 5000 :
        return 9     #주문가능
    else :
        return 5     #주문불가

#macd구하기
def get_signal(ticker):
    df = pybithumb.get_candlestick(ticker, chart_intervals="10m")

    macd3 = sum(df["close"].tail(3)) / 3
    macd8 = sum(df["close"].tail(8)) / 8
    macd = macd3 - macd8
    macd15 = macd3
    macd40 = macd8

    for i in range(1,5):
        macd15 = macd15 + (sum(df["close"].tail(i+3)) - sum(df["close"].tail(i)))/3
        macd40 = macd40 + (sum(df["close"].tail(i+8)) - sum(df["close"].tail(i)))/8

    macd15 = macd15 / 5
    macd40 = macd40 / 5

    rsi = macd15 - macd40
    macdsig = macd - rsi

    if macdsig > 0 :
        return 1               #매수신호
    elif macdsig <= 0 :
        return 2               #매도신호

# 자동주문
while True:
    try :
        now = datetime.datetime.now()
        if now.minute == 0 and now.second == 0 or now.minute == 10 and now.second == 0 or now.minute == 20 and now.second == 0 or now.minute == 30 and now.second == 0 or now.minute == 40 and now.second == 0 or now.minute == 50 and now.second == 0 :
            status = get_status("BTC")
            if status == 9 :
                signal = get_signal("BTC")
                if signal == 1 :
                    buy_crypto_currency("BTC")
            elif status == 5 and bithumb.get_balance("BTC")[0] != 0:
                if signal == 2 :
                    sell_crypto_currency("BTC")

    except :
        pass
    time.sleep(0.98)
