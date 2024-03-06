<p align="center">
    <img src="assets/banner.png" width="55%" alt="Gemini Banner" align="center">
</p>
<p align="center">
    <a href="https://pypi.org/project/gemini-webapi">
        <img src="https://img.shields.io/pypi/v/gemini-webapi" alt="PyPI">
    </a>
    <a href="https://pepy.tech/project/gemini-webapi">
        <img src="https://static.pepy.tech/badge/gemini-webapi" alt="Downloads">
    </a>
    <a href="https://github.com/HanaokaYuzu/Gemini-API/network/dependencies">
        <img src="https://img.shields.io/librariesio/github/HanaokaYuzu/Gemini-API" alt="Dependencies">
    </a>
    <a href="https://github.com/HanaokaYuzu/Gemini-API/blob/master/LICENSE">
        <img src="https://img.shields.io/github/license/HanaokaYuzu/Gemini-API" alt="License">
    </a>
</p>
<p align="center">
    <a href="https://star-history.com/#HanaokaYuzu/Gemini-API">
        <img src="https://img.shields.io/github/stars/HanaokaYuzu/Gemini-API?style=social" alt="GitHub stars">
    </a>
    <a href="https://github.com/HanaokaYuzu/Gemini-API/issues">
        <img src="https://img.shields.io/github/issues/HanaokaYuzu/Gemini-API?style=social&logo=github" alt="GitHub issues">
    </a>
    <a href="https://github.com/HanaokaYuzu/Gemini-API/actions/workflows/pypi-publish.yml">
        <img src="https://github.com/HanaokaYuzu/Gemini-API/actions/workflows/pypi-publish.yml/badge.svg" alt="CI">
    </a>
</p>

# <img src="assets/logo.svg" width="35px" alt="Gemini Icon" /> Gemini-API

A reverse-engineered asynchronous python wrapper for [Google Gemini](https://gemini.google.com) web chat (formerly Bard).

## Features

- **ImageFx Support** - Supports retrieving images generated by ImageFx, Google's latest AI image generator.
- **Extension Support** - Supports generating contents with [Gemini extensions](https://gemini.google.com/extensions) on, like YouTube and Gmail.
- **Classified Outputs** - Auto categorizes texts, web images and AI generated images from the response.
- **Official Flavor** - Provides a simple and elegant interface inspired by [Google Generative AI](https://ai.google.dev/tutorials/python_quickstart)'s official API.
- **Asynchronous** - Utilizes `asyncio` to run generating tasks and return outputs efficiently.

## Table of Contents

- [Features](#features)
- [Table of Contents](#table-of-contents)
- [Installation](#installation)
- [Authentication](#authentication)
- [Usage](#usage)
  - [Initialization](#initialization)
  - [Generate contents from text inputs](#generate-contents-from-text-inputs)
  - [Conversations across multiple turns](#conversations-across-multiple-turns)
  - [Retrieve images in response](#retrieve-images-in-response)
  - [Generate images with ImageFx](#generate-images-with-imagefx)
  - [Save images to local files](#save-images-to-local-files)
  - [Generate contents with Gemini extensions](#generate-contents-with-gemini-extensions)
  - [Check and switch to other reply candidates](#check-and-switch-to-other-reply-candidates)
- [References](#references)
- [Stargazers](#stargazers)

## Installation

```bash
pip install gemini_webapi
```

## Authentication

- Go to <https://gemini.google.com> and login with your Google account
- Press F12 for web inspector, go to `Network` tab and refresh the page
- Click any request and copy cookie values of `__Secure-1PSID` and `__Secure-1PSIDTS`

> [!TIP]
> `__Secure-1PSIDTS` could get expired frequently if <https://gemini.google.com> is kept opened in the browser after copying cookies. It's recommended to get cookies from a separate session (e.g. a new login in browser's private mode) if you are building a keep-alive service with this package.
>
> For more details, please refer to discussions in [issue #6](https://github.com/HanaokaYuzu/Gemini-API/issues/6)

## Usage

### Initialization

Import required packages and initialize a client with your cookies obtained from the previous step.

```python
import asyncio
from gemini_webapi import GeminiClient

# Replace "COOKIE VALUE HERE" with your actual cookie values
Secure_1PSID = "COOKIE VALUE HERE"
Secure_1PSIDTS = "COOKIE VALUE HERE"

async def main():
    client = GeminiClient(Secure_1PSID, Secure_1PSIDTS, proxy=None)
    await client.init(timeout=30, auto_close=False, close_delay=300)

asyncio.run(main())
```

> [!TIP]
> `auto_close` and `close_delay` are optional arguments for automatically closing the client after a certain period of inactivity. This feature is disabled by default. In a keep-alive service like chatbot, it's recommended to set `auto_close` to `True` combined with reasonable seconds of `close_delay` for better resource management.

### Generate contents from text inputs

Ask a one-turn quick question by calling `GeminiClient.generate_content`.

```python
async def main():
    response = await client.generate_content("Hello World!")
    print(response.text)

asyncio.run(main())
```

> [!TIP]
> Simply use `print(response)` to get the same output if you just want to see the response text

### Conversations across multiple turns

If you want to keep conversation continuous, please use `GeminiClient.start_chat` to create a `ChatSession` object and send messages through it. The conversation history will be automatically handled and get updated after each turn.

```python
async def main():
    chat = client.start_chat()
    response1 = await chat.send_message("Briefly introduce Europe")
    response2 = await chat.send_message("What's the population there?")
    print(response1.text, response2.text, sep="\n\n----------------------------------\n\n")

asyncio.run(main())
```

### Retrieve images in response

Images in the API's output are stored as a list of `Image` objects. You can access the image title, URL, and description by calling `image.title`, `image.url` and `image.alt` respectively.

```python
async def main():
    response = await client.generate_content("Send me some pictures of cats")
    for image in response.images:
        print(image, "\n\n----------------------------------\n")

asyncio.run(main())
```

### Generate images with ImageFx

In February 2022, Google introduced a new AI image generator called ImageFx and integrated it into Gemini. You can ask Gemini to generate images with ImageFx simply by natural language.

> [!IMPORTANT]
> Google has some limitations on the image generation feature in Gemini, so its availability could be different per region/account. Here's a summary copied from [official documentation](https://support.google.com/gemini/answer/14286560) (as of February 15th, 2024):
>
> > Image generation in Gemini Apps is available in most countries, except in the European Economic Area (EEA), Switzerland, and the UK. It’s only available for **English prompts**.
> >
> > This feature’s availability in any specific Gemini app is also limited to the supported languages and countries of that app.
> >
> > For now, this feature isn’t available to users under 18.

```python
async def main():
    response = await client.generate_content("Generate some pictures of cats")
    for image in response.images:
        print(image, "\n\n----------------------------------\n")

asyncio.run(main())
```

> [!NOTE]
> by default, when asked to send images (like the previous example), Gemini will send images fetched from web instead of generating images with AI model, unless you specifically require to "generate" images in your prompt. In this package, web images and generated images are treated differently as `WebImage` and `GeneratedImage`, and will be automatically categorized in the output.

### Save images to local files

You can save images returned from Gemini to local files under `/temp` by calling `Image.save()`. Optionally, you can specify the file path and file name by passing `path` and `filename` arguments to the function and skip images with invalid file names by passing `skip_invalid_filename=True`. Works for both `WebImage` and `GeneratedImage`.

```python
async def main():
    response = await client.generate_content("Generate some pictures of cats")
    for i, image in enumerate(response.images):
        await image.save(path="temp/", filename=f"cat_{i}.png", verbose=True)

asyncio.run(main())
```

### Generate contents with Gemini extensions

> [!IMPORTANT]
> To access Gemini extensions in API, you must activate them on the [Gemini website](https://gemini.google.com/extensions) first. Same as image generation, Google also has limitations on the availability of Gemini extensions. Here's a summary copied from [official documentation](https://support.google.com/gemini/answer/13695044) (as of February 18th, 2024):
>
> > To use extensions in Gemini Apps:
> >
> > Sign in with your personal Google Account that you manage on your own. Extensions, including the Google Workspace extension, are currently not available to Google Workspace accounts for school, business, or other organizations.
> >
> > Have Gemini Apps Activity on. Extensions are only available when Gemini Apps Activity is turned on.
> >
> > Important: For now, extensions are available in **English, Japanese, and Korean** only.

After activating extensions for your account, you can access them in your prompts either by natural language or by starting your prompt with "@" followed by the extension keyword.

```python
async def main():
    response1 = await client.generate_content("@Gmail What's the latest message in my mailbox?")
    print(response1, "\n\n----------------------------------\n")

    response2 = await client.generate_content("@Youtube What's the lastest activity of Taylor Swift?")
    print(response2, "\n\n----------------------------------\n")

asyncio.run(main())
```

> [!NOTE]
> For the available regions limitation, it actually only requires your Google account's **preferred language** to be set to one of the three supported languages listed above. You can change your language settings [here](https://myaccount.google.com/language).

### Check and switch to other reply candidates

A response from Gemini usually contains multiple reply candidates with different generated contents. You can check all candidates and choose one to continue the conversation. By default, the first candidate will be chosen automatically.

```python
async def main():
    # Start a conversation and list all reply candidates
    chat = client.start_chat()
    response = await chat.send_message("Recommend a science fiction book for me.")
    for candidate in response.candidates:
        print(candidate, "\n\n----------------------------------\n")

    if len(response.candidates) > 1:
        # Control the ongoing conversation flow by choosing candidate manually
        new_candidate = chat.choose_candidate(index=1)  # Choose the second candidate here
        followup_response = await chat.send_message("Tell me more about it.")  # Will generate contents based on the chosen candidate
        print(new_candidate, followup_response, sep="\n\n----------------------------------\n\n")
    else:
        print("Only one candidate available.")

asyncio.run(main())
```

## References

[Google AI Studio](https://ai.google.dev/tutorials/ai-studio_quickstart)

[acheong08/Bard](https://github.com/acheong08/Bard)

## Stargazers

<p align="center">
    <a href="https://star-history.com/#HanaokaYuzu/Gemini-API">
        <img src="https://api.star-history.com/svg?repos=HanaokaYuzu/Gemini-API&type=Date" width="75%" alt="Star History Chart">
    </a>
</p>
