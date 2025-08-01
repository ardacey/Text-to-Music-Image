# Dynamic Narrative to Multimedia API

This project is an agent-based API built with **FastAPI** and **LangGraph** that dynamically generates coherent visual scenes and adaptive musical scores from short narrative descriptions. It's designed for applications like games or interactive stories where multimedia assets need to be created on the fly in response to an evolving story.

The key feature is its ability to maintain **visual consistency** across a sequence of generations by using the previously generated image as a base for the next one (img2img), creating a fluid, story-like progression.

## ✨ Features

-   **Agentic Parallel Workflow**: Utilizes LangGraph to orchestrate two parallel agentic chains—one for image generation and one for music generation—that work from the same narrative input.
-   **Evolving Visuals (Image-to-Image Continuity)**: The `ArtDirector` agent creates a prompt, and the `ImageGenerator` uses **Stable Diffusion 3.5 Large** with an img2img pipeline. Each new image uses the previous one as a starting point, ensuring a smooth visual narrative.
-   **Adaptive Music Generation**: The `MusicTheorist` agent analyzes the narrative's mood to generate a fitting musical prompt, which is then composed into a track by the **ACE-Step** model.
-   **High-Performance API**: Built with FastAPI, providing a robust and scalable endpoint for generating assets.
-   **Asynchronous & Efficient**: Models are pre-loaded and warmed up during the application's lifespan startup event for faster response times on the first request. The API also asynchronously handles downloading previous images for the img2img process.
-   **Colab-Ready**: Designed to run seamlessly in a Google Colab environment with GPU acceleration.

## 🏛️ Architecture: The Parallel Agentic Graph

The system's core is a LangGraph workflow where the art and music generation processes run in parallel, triggered by a single narrative input.



1.  **START (Input)**: The graph receives the initial state, containing the `narrative`, `characters`, and optionally, the `previous_image_path` and `previous_image_style`.
2.  **Parallel Execution**: The graph simultaneously kicks off two independent chains:
    -   **Art Chain**:
        -   **`ArtDirector`**: Analyzes the narrative and the *style of the previous image* to create a new, concise image prompt and a style description for the *next* generation, ensuring visual consistency.
        -   **`ImageGenerator`**: Takes the prompt and the previous image. It uses an img2img pipeline to generate a new frame that logically follows the last one.
    -   **Music Chain**:
        -   **`MusicTheorist`**: Analyzes the narrative to determine the appropriate mood, genre, tempo, and instruments.
        -   **`MusicGenerator`**: Takes the music theory prompt and synthesizes an audio track.
3.  **END (Output)**: The graph waits for both chains to complete and then aggregates the results, providing URLs to the newly created image and music files.

## 🛠️ Technology Stack

-   **API Framework**: [FastAPI](https://fastapi.tiangolo.com/)
-   **Orchestration**: [LangGraph](https://github.com/langchain-ai/langgraph)
-   **LLM (Analysis & Prompting)**: [Google Gemini](https://ai.google) via `langchain_google_genai`
-   **Image Generation Model**: [Stable Diffusion 3.5 Large](https://huggingface.co/stabilityai/stable-diffusion-3.5-large) via `diffusers`
-   **Music Generation Model**: [ACE-Step](https://github.com/ace-step/ACE-Step)
-   **Deployment**: Google Colab, `uvicorn`, `pyngrok`

## 🚀 Setup and Usage

This project is optimized for a Google Colab environment with a GPU (A100 recommended for SD 3.5 Large).

### Prerequisites
-   A Google Account
-   Git

### Running in Google Colab

1.  **Open the Notebook**: Open the `text_to_music_image.ipynb` notebook in Google Colab.

2.  **Set Up API Keys & Tokens**: You will need keys/tokens for Google Gemini, Hugging Face, and ngrok.
    -   Get a **Google Gemini API Key** from [Google AI Studio](https://aistudio.google.com/app/apikey).
    -   Get a **Hugging Face User Access Token** from your [Hugging Face settings](https://huggingface.co/settings/tokens) with at least `read` permissions. This is required to download the Stable Diffusion model.
    -   Get an **ngrok Authtoken** by signing up at the [ngrok Dashboard](https://dashboard.ngrok.com).

3.  **Add Keys to Colab Secrets**:
    -   In your Colab notebook, click the "🔑" (Secrets) icon in the left sidebar.
    -   Add three new secrets:
        -   `GEMINI_API_KEY`: Paste your Google Gemini key.
        -   `HF_TOKEN`: Paste your Hugging Face token.
        -   `NGROK_AUTH_TOKEN`: Paste your ngrok authtoken.
    -   Ensure the "Notebook access" toggle is enabled for all secrets.

4.  **Run the Cells**:
    -   Execute the cells in the notebook sequentially.
    -   The cells will install dependencies, set up the project, and start the FastAPI server using `uvicorn`. The server will pre-load and compile the models, which may take several minutes.
    -   The final cell will start the server and provide a public `ngrok` URL for your API.

### Testing the API

Once the server is running, you can interact with it using any HTTP client, like `curl`.

**Endpoint**: `POST /generate`

**Request Body**:
```json
{
  "narrative": "The knight, weary from battle, enters a glowing, mystical forest. The ancient trees hum with a soft, ethereal energy.",
  "characters": ["A weary knight in battered silver armor"],
  "previous_image_url": "URL_TO_PREVIOUS_IMAGE or null for the first request",
  "previous_image_style": "STYLE_DESCRIPTION_FROM_PREVIOUS_RESPONSE or null for the first request"
}
```

**Response Body**:
```json
{
  "image_url": "https://<id>.ngrok-free.app/outputs/images/image_....png",
  "music_url": "https://<id>.ngrok-free.app/outputs/music/music_....wav",
  "new_image_style": "ethereal fantasy digital painting, glowing magical flora"
}
```
> **Note**: For subsequent calls, use the `image_url` from the previous response as the `previous_image_url` and the `new_image_style` as the `previous_image_style` to maintain continuity.

**Example `curl` Command:**

```bash
# Replace <YOUR_NGROK_URL> with the URL from your Colab output.
# For the first request, previous_image_url and previous_image_style are null.
curl -X POST "<YOUR_NGROK_URL>/generate" \
-H "Content-Type: application/json" \
-d '{
  "narrative": "A lone warrior stands on a cliff overlooking a stormy sea, her cloak billowing in the wind.",
  "characters": ["A female warrior with a determined expression"],
  "previous_image_url": null,
  "previous_image_style": null
}'
```

## 📁 Project Structure

```
/
├── text_to_music_image.ipynb   # Main Google Colab notebook.
├── api.py                      # FastAPI application logic.
├── ACE-Step/                   # Cloned repository for the music model.
├── outputs/
│   ├── images/                 # Generated images are saved here.
│   └── music/                  # Generated music is saved here.
├── temp_downloads/             # For temporary files like downloaded previous images.
└── src/
    ├── graph.py                # LangGraph workflow definition.
    ├── models.py               # Pydantic models (ArtConcept, MusicTheory).
    ├── state.py                # LangGraph state definition.
    └── nodes/
        ├── art_director.py
        ├── image_generator.py
        ├── music_theorist.py
        └── music_generator.py
```
