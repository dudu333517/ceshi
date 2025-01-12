import requests
import time

def load_token_addresses(filename):
    tokens = {}
    try:
        with open(filename, 'r') as file:
            for line in file:
                line = line.strip()
                if line:
                    token_info = line.split(":")
                    if len(token_info) == 2:
                        token_name = token_info[0].strip()
                        token_address = token_info[1].strip()
                        tokens[token_name] = token_address
    except FileNotFoundError:
        print(f"\u6587\u4ef6 {filename} \u672a\u627e\u5230\u3002")
    except Exception as e:
        print(f"\u8bfb\u53d6\u6587\u4ef6\u65f6\u51fa\u9519: {e}")
    return tokens

def check_price_difference(prices, token_name):
    if len(prices) < 2:
        print(f"{token_name} \u7684\u4ef7\u683c\u5dee\u5f02\u65e0\u6cd5\u6bd4\u8f83\uff0c\u5c11\u4e8e\u4e24\u4e2a\u4ea4\u6613\u5bf9\u3002")
        return

    # 去除流动性小于 1000 的数据
    filtered_prices = [entry for entry in prices if entry[2] >= 1000]

    if len(filtered_prices) < 2:
        print(f"{token_name} \u7684\u6709\u6548\u4ef7\u683c\u6570\u636e\u4e0d\u8db3\u4ee5\u6bd4\u8f83 (\u6d41\u52a8\u6027\u5927\u4e8e\u7b49\u4e8e 1000)。")
        return

    max_chain, max_price, max_liquidity = max(filtered_prices, key=lambda x: x[1])
    min_chain, min_price, min_liquidity = min(filtered_prices, key=lambda x: x[1])
    price_diff_percentage = ((max_price - min_price) / min_price) * 100

    # 根据流动性判断利润差异报警
    if min_liquidity < 12000 and price_diff_percentage > 8:
        print(f"\u26a0\ufe0f \u8b66\u62a5: {token_name} \u4e0d\u540c\u94fe\u4e4b\u95f4\u7684\u4ef7\u683c\u5dee\u5f02\u8d85\u8fc7 8%\uff0c\u4e14\u6d41\u52a8\u6027 < 12000\u3002")
        print(f"\u6700\u5927\u4ef7\u683c: {max_price} ({max_chain}, \u6d41\u52a8\u6027: {max_liquidity}), \u6700\u5c0f\u4ef7\u683c: {min_price} ({min_chain}, \u6d41\u52a8\u6027: {min_liquidity}), \u5dee\u5f02: {price_diff_percentage:.2f}%")
    elif min_liquidity >= 12000 and price_diff_percentage > 4:
        print(f"\u26a0\ufe0f \u8b66\u62a5: {token_name} \u4e0d\u540c\u94fe\u4e4b\u95f4\u7684\u4ef7\u683c\u5dee\u5f02\u8d85\u8fc7 4%\uff0c\u4e14\u6d41\u52a8\u6027 \u2265 12000\u3002")
        print(f"\u6700\u5927\u4ef7\u683c: {max_price} ({max_chain}, \u6d41\u52a8\u6027: {max_liquidity}), \u6700\u5c0f\u4ef7\u683c: {min_price} ({min_chain}, \u6d41\u52a8\u6027: {min_liquidity}), \u5dee\u5f02: {price_diff_percentage:.2f}%")
    else:
        print(f"{token_name} \u4ef7\u683c\u5dee\u5f02\u5728\u53ef\u63a5\u53d7\u8303\u56f4\u5185\u3002")

    # 输出流动性最小的链
    #min_liquidity_chain, _, min_liquidity_value = min(filtered_prices, key=lambda x: x[2])
    #print(f"\u6d41\u52a8\u6027\u6700\u5c0f\u7684\u94fe\u4e3a: {min_liquidity_chain}\uff0c\u6d41\u52a8\u6027: {min_liquidity_value}")

def process_token_data(token_name, token_address):
    url = f"https://api.dexscreener.com/latest/dex/tokens/{token_address}"
    excluded_chains = ["ethereum", "elastos"]
    try:
        response = requests.get(url, timeout=10)  # 设置请求超时为10秒
        response.raise_for_status()
        data = response.json()

        chain_prices = {}

        pairs = data.get("pairs", [])
        if not pairs:
            print(f"\u6ca1\u6709\u627e\u5230 {token_name} \u7684\u4ea4\u6613\u5bf9\u6570\u636e\u3002")
        else:
            for pair in pairs:
                chain = pair.get("chainId", "\u672a\u77e5\u94fe")
                if chain in excluded_chains:
                    continue

                price_usd = pair.get("priceUsd", "N/A")
                volume_24h = pair.get("volume", {}).get("h24", 0)
                dex = pair.get("dexId", "\u672a\u77e5DEX")
                liquidity_usd = pair.get("liquidity", {}).get("usd", 0)

                if price_usd != "N/A" and volume_24h > 50:
                    if chain not in chain_prices or volume_24h > chain_prices[chain]["volume"]:
                        chain_prices[chain] = {
                            "price_usd": float(price_usd),
                            "volume": volume_24h,
                            "dex": dex,
                            "liquidity": liquidity_usd
                        }

            if chain_prices:
                prices = [(chain, info["price_usd"], info["liquidity"]) for chain, info in chain_prices.items()]
                check_price_difference(prices, token_name)
            else:
                print(f"\u6ca1\u6709\u627e\u5230 {token_name} \u7684\u6709\u6548\u4ef7\u683c\u6570\u636e\uff08\u7b26\u5408\u4ea4\u6613\u91cf > 10 \u7684\u6761\u76ee\uff09\u3002")
    except requests.exceptions.RequestException as e:
        print(f"\u8bf7\u6c42 {token_name} \u7684\u6570\u636e\u5931\u8d25: {e}")
    except ValueError:
        print(f"\u89e3\u6790 {token_name} \u7684 JSON \u6570\u636e\u65f6\u51fa\u9519\u3002")
    except Exception as e:
        print(f"\u5904\u7406 {token_name} \u6570\u636e\u65f6\u53d1\u751f\u672a\u77e5\u9519\u8bef: {e}")

def main():
    filename = "tokens.txt"
    max_requests_per_minute = 250

    tokens = load_token_addresses(filename)
    if not tokens:
        print("\u6ca1\u6709\u52a0\u8f7d\u5230\u4ee3\u5e01\u6570\u636e\uff0c\u9000\u51fa\u7a0b\u5e8f\u3002")
        return

    start_time = time.time()
    request_count = 0

    while True:
        for token_name, token_address in tokens.items():
            elapsed_time = time.time() - start_time
            if elapsed_time < 60:
                if request_count < max_requests_per_minute:
                    process_token_data(token_name, token_address)
                    request_count += 1
                else:
                    sleep_time = 60 - elapsed_time
                    print(f"\u8bf7\u6c42\u6b21\u6570\u8fbe\u5230\u9650\u5236\uff0c\u7b49\u5f85 {sleep_time:.2f} \u79d2...")
                    time.sleep(sleep_time)
                    start_time = time.time()
                    request_count = 0
            else:
                start_time = time.time()
                request_count = 0

            time.sleep(0.5)

if __name__ == "__main__":
    main()
