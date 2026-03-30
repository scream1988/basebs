# basebsimport time
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

# bytecode size threshold (bytes)
SIZE_THRESHOLD = 20000


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Detecting large contract deployments...\n")

    last_block = w3.eth.block_number

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block > last_block:

                for block_num in range(last_block + 1, current_block + 1):

                    block = w3.eth.get_block(block_num, full_transactions=True)

                    for tx in block.transactions:

                        if tx.to is None:  # contract creation
                            receipt = w3.eth.get_transaction_receipt(tx.hash)

                            if receipt.contractAddress:
                                code = w3.eth.get_code(receipt.contractAddress)
                                size = len(code)

                                if size >= SIZE_THRESHOLD:
                                    print("📦 Large Contract Deployed")
                                    print("Block:", block_num)
                                    print("Contract:", receipt.contractAddress)
                                    print("Bytecode size:", size, "bytes")
                                    print("Deployer:", tx["from"])
                                    print("Tx:", tx.hash.hex())
                                    print()

                last_block = current_block

            time.sleep(2)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()
