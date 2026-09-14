import os
import json
import requests

from TikTokLive import TikTokLiveClient
from TikTokLive.events import CommentEvent, GiftEvent, LikeEvent, FollowEvent, ShareEvent

TIKTOK_USERNAME = os.environ["TIKTOK_USERNAME"]
ROBLOX_API_KEY = os.environ["ROBLOX_API_KEY"]
ROBLOX_UNIVERSE_ID = "105768446109772"
ROBLOX_TOPIC = "TikTokLive_GulDana"

client = TikTokLiveClient(unique_id=TIKTOK_USERNAME)


def send_to_roblox(data):
    url = f"https://apis.roblox.com/cloud/v2/universes/{ROBLOX_UNIVERSE_ID}:publishMessage"

    payload = {
        "topic": ROBLOX_TOPIC,
        "message": json.dumps(data, ensure_ascii=False)
    }

    response = requests.post(
        url,
        headers={
            "x-api-key": ROBLOX_API_KEY,
            "Content-Type": "application/json"
        },
        json=payload,
        timeout=10
    )

    print("ROBLOX:", response.status_code, response.text)


@client.on("connect")
async def on_connect(event):
    print("CONNECTED TO TIKTOK LIVE:", event.unique_id)


@client.on("comment")
async def on_comment(event: CommentEvent):
    send_to_roblox({
        "type": "comment",
        "username": event.user.unique_id,
        "comment": event.comment
    })


@client.on("gift")
async def on_gift(event: GiftEvent):
    if event.gift is None:
        return

    send_to_roblox({
        "type": "gift",
        "username": event.user.unique_id,
        "giftName": event.gift.name,
        "amount": event.repeat_count
    })


@client.on("like")
async def on_like(event: LikeEvent):
    send_to_roblox({
        "type": "like",
        "username": event.user.unique_id,
        "amount": getattr(event, "count", 1)
    })


@client.on("follow")
async def on_follow(event: FollowEvent):
    send_to_roblox({
        "type": "follow",
        "username": event.user.unique_id
    })


@client.on("share")
async def on_share(event: ShareEvent):
    send_to_roblox({
        "type": "share",
        "username": event.user.unique_id
    })


if __name__ == "__main__":
    print("TIKTOK → ROBLOX BRIDGE STARTING...")
    client.run()
