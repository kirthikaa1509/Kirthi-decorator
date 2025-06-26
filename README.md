# Kirthi-decorator
import threading
import asyncio
import aiohttp
import time

# Shared log and lock
download_log = []
log_lock = threading.Lock()

# 🔹 Async function to fetch a URL
async def fetch_url(session, url):
    async with session.get(url) as response:
        text = await response.text()
        return url, len(text)  # Return length for simulation

# 🔹 Async wrapper to handle multiple URL fetches
async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        return await asyncio.gather(*tasks)

# 🔹 Thread task: run asyncio in each thread
def threaded_fetch(name, urls):
    print(f"[{name}] Starting download...")

    # Run async fetch in this thread
    results = asyncio.run(fetch_all(urls))

    with log_lock:
        for url, length in results:
            download_log.append((name, url, length))
            print(f"[{name}] Fetched {url} ({length} chars)")

    print(f"[{name}] Completed.\n")

# 🔹 Main thread
if __name__ == "__main__":
    # Sample URLs
    urls_set1 = ["https://httpbin.org/get", "https://httpbin.org/uuid"]
    urls_set2 = ["https://httpbin.org/headers", "https://httpbin.org/ip"]

    # Create threads
    t1 = threading.Thread(target=threaded_fetch, args=("Worker-1", urls_set1))
    t2 = threading.Thread(target=threaded_fetch, args=("Worker-2", urls_set2))

    start_time = time.time()

    t1.start()
    t2.start()
    t1.join()
    t2.join()

    end_time = time.time()

    # Final output
    print("📝 Download Log:")
    for entry in download_log:
        name, url, length = entry
        print(f" - {name} got {url} with {length} chars")

    print(f"\n⏱ Total time taken: {end_time - start_time:.2f} seconds")
