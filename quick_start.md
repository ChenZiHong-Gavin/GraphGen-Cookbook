# Quick Start

Experience it on the [OpenXLab Application Center](https://g-app-center-000704-6802-aerppvq.openxlab.space) and [FAQ](https://github.com/open-sciencelab/GraphGen/issues/10).

### Gradio Demo

   ```bash
   python webui/app.py
   ```

![ui](https://github.com/user-attachments/assets/3024e9bc-5d45-45f8-a4e6-b57bd2350d84)

### Run from PyPI

1. Install GraphGen
   ```bash
   pip install graphg
   ```

2. Run in CLI
   ```bash
   SYNTHESIZER_MODEL=your_synthesizer_model_name \
   SYNTHESIZER_BASE_URL=your_base_url_for_synthesizer_model \
   SYNTHESIZER_API_KEY=your_api_key_for_synthesizer_model \
   TRAINEE_MODEL=your_trainee_model_name \
   TRAINEE_BASE_URL=your_base_url_for_trainee_model \
   TRAINEE_API_KEY=your_api_key_for_trainee_model \
   graphg --output_dir cache
   ```

### Run from Source

1. Install dependencies
    ```bash
    pip install -r requirements.txt
    ```
2. Configure the environment
   - Create an `.env` file in the root directory
     ```bash
     cp .env.example .env
     ```
   - Set the following environment variables:
     ```bash
     # Synthesizer is the model used to construct KG and generate data
     SYNTHESIZER_MODEL=your_synthesizer_model_name
     SYNTHESIZER_BASE_URL=your_base_url_for_synthesizer_model
     SYNTHESIZER_API_KEY=your_api_key_for_synthesizer_model
     # Trainee is the model used to train with the generated data
     TRAINEE_MODEL=your_trainee_model_name
     TRAINEE_BASE_URL=your_base_url_for_trainee_model
     TRAINEE_API_KEY=your_api_key_for_trainee_model
     ```
3. (Optional) If you want to modify the default generated configuration, you can edit the content of the configs/graphgen_config.yaml file.
    ```yaml
    # configs/graphgen_config.yaml
    # Example configuration
    data_type: "raw"
    input_file: "resources/examples/raw_demo.jsonl"
    # more configurations...
    ```
4. Run the generation script
   ```bash
   bash scripts/generate.sh
   ```
5. Get the generated data
   ```bash
   ls cache/data/graphgen
   ```
