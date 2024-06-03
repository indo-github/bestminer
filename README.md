
import time
from binance.client import Client
import pandas as pd
import schedule

# API keys dari Binance
api_key = 'YLNFzshoAXGt0lDhtvVzg7LFBcQwccjzwVUUbgj1yWwHaycF5QQ2gFWkbhaYnYCx'
api_secret = '1SOGi8KxOcJGw5ZXWiKONWpFyl3qFYJmDAGznbpma9OIie3bT17PsASFWE7SIcdB'

# Inisialisasi client
client = Client(api_key, api_secret)

# Fungsi untuk mendapatkan data candle
def get_candle_data(symbol, interval, limit=1):
    klines = client.get_klines(symbol=symbol, interval=interval, limit=limit)
    return pd.DataFrame(klines, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume', 'close_time', 
                                         'quote_asset_volume', 'number_of_trades', 'taker_buy_base_asset_volume', 
                                         'taker_buy_quote_asset_volume', 'ignore'])

# Fungsi untuk menempatkan order SELL STOP
def place_sell_stop(symbol, quantity, price):
    try:
        order = client.create_order(
            symbol=symbol,
            side='SELL',
            type='STOP_LOSS_LIMIT',
            quantity=quantity,
            price=str(price),
            stopPrice=str(price),
            timeInForce='GTC'
        )
        return order
    except Exception as e:
        print(f"An error occurred: {e}")
        return None

# Fungsi utama untuk trading logic
def trading_bot():
    symbol = 'SHIBUSDT'
    
    # Mendapatkan data candle 4 jam terakhir
    candle_4h = get_candle_data(symbol, Client.KLINE_INTERVAL_4HOUR)
    candle_4h_close = float(candle_4h['close'].iloc[0])
    candle_4h_open = float(candle_4h['open'].iloc[0])
    candle_4h_low = float(candle_4h['low'].iloc[0])
    
    # Hitung persentase perubahan harga
    percent_change_4h = ((candle_4h_close - candle_4h_open) / candle_4h_open) * 100
    
    # Logika untuk order SELL pertama
    total_shib = 1000000  # Misal total SHIB yang dimiliki
    if abs(percent_change_4h) > 1:
        volume_5_percent = total_shib * 0.05
        place_sell_stop(symbol, volume_5_percent, candle_4h_low)
    
    # Logika untuk order SELL lanjutan dan Take Profit
    # Implementasikan sesuai dengan detail yang diberikan
    
    # Misal implementasi fungsi untuk mendapatkan data candle 5 menit
    def get_candle_data_5m(symbol):
        return get_candle_data(symbol, Client.KLINE_INTERVAL_5MINUTE)

# Menjadwalkan bot berjalan setiap 4 jam
schedule.every(4).hours.do(trading_bot)

while True:
    schedule.run_pending()
    time.sleep(1)
