# RADAR_AI_Detection
Code for our NeurIPS2023 accepted paper: [RADAR: Robust AI-Text Detection via Adversarial Learning](https://proceedings.neurips.cc/paper_files/paper/2023/file/30e15e5941ae0cdab7ef58cc8d59a4ca-Paper-Conference.pdf).
We tested RADAR on 8 LLMs including Vicuna and LLaMA. The results shown that RADAR can attain good detection performance on LLM-generated AI-text while robust against paraphrasing.
## Environment Build
```bash
    conda env create -f radar_core.yaml 
    # to init a environment with packages installed using conda
    pip install -r radar_env_pip.txt 
    # to install packages install using pip
```
## Use RADAR to get AI-generated probability
Our RADAR detector is trained from RoBERTa-large model. You can use it as using RoBERTa-large model. Here is an example to use RADAR to get the probbaility that the text is generated by Vicuna.

```python
detector = transformers.AutoModelForSequenceClassification.from_pretrained("TrustSafeAI/RADAR-Vicuna-7B")
tokenizer = transformers.AutoTokenizer.from_pretrained("TrustSafeAI/RADAR-Vicuna-7B")
detector.eval()
detector.to(device)
Text_Input=["I'm not a chatbot"]
with torch.no_grad():
  inputs = tokenizer(Text_input, padding=True, truncation=True, max_length=512, return_tensors="pt")
  inputs = {k:v.to(device) for k,v in inputs.items()}
  output_probs = F.log_softmax(detector(**inputs).logits,-1)[:,0].exp().tolist()
  print("Probability of AI-generated texts is",output_probs)
```

## Paraphrase the ai-text to evade detection
We prompt the gpt-3.5-turbo/gpt-4 to paraphrase the ai-generated text to make it more like human-written.

```python
import openai
openai.api_key = "your_api_key"
def _openai_response(text,openai_model):
    # get paraphrases of text from the openai model
    # openai_model can be gpt-3.5-turbo/gpt-4
    system_instruct = {"role": "system", "content": "Enhance the word choices in the sentence to sound more like that of a human."}
    user_input={"role": "user", "content": text}
    messages = [system_instruct,user_input]
    k_wargs = { "messages":messages, "model": openai_model}
    r = openai.ChatCompletion.create(**k_wargs)['choices'][0].message.content
    return r 

```

## Calculate the Detection AUROC
We may need to calcalute the detction auroc of the detector.
```python
from sklearn.metrics import auc,roc_curve
def get_roc_metrics(human_preds, ai_preds):
    # human_preds is the ai-generated probabiities of human-text
    # ai_preds is the ai-generated probabiities of human-text
    fpr, tpr, _ = roc_curve([0] * len(human_preds) + [1] * len(ai_preds), human_preds + ai_preds,pos_label=1)
    roc_auc = auc(fpr, tpr)
    return fpr.tolist(), tpr.tolist(), float(roc_auc)
```

## Examples

We provides some examples of using RADAR in [radar_examples.ipynb](./radar_examples.ipynb). You can refer to it to get more famiiar with RADAR working flow.

## Citation

If you find RADAR useful, please cite the following paper:
```bash
@inproceedings{DBLP:conf/nips/HuCH23,
  author       = {Xiaomeng Hu and
                  Pin{-}Yu Chen and
                  Tsung{-}Yi Ho},
  title        = {{RADAR:} Robust AI-Text Detection via Adversarial Learning},
  booktitle    = {Advances in Neural Information Processing Systems 36: Annual Conference
                  on Neural Information Processing Systems 2023, NeurIPS 2023, New Orleans,
                  LA, USA, December 10 - 16, 2023},
  year         = {2023}
}
```
## Contact

Feel free to contact [Xiaomeng Hu](mailto:greghxm@foxmail.com) if you have any questions.
