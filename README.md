# cogvlm-cogagent

CogVLM
VLM SOTA 중 하나
github.com/THUDM…
1. shallow alignment method
파라미터가 동결된 미리 학습된 vision encoder와 언어모델 연결
(linear layer 또는 Q-former로 연결)
-> llm의 임베딩 벡터에 image features이 결합된다
위의 과정은 image 데이터와 text 데이터의 강한 결합을 이루어내지 못함 -> 시각적 feature 가 기존의 다수의 layer를 통과하며 약해짐
PaLI( 구글-ScreenAI ), Qwen-VL(알리바바)는 이를 direct training, 동결 안시키고 처음부터 다 학습시킴(SFT, pretraining)
위의 방법 문제: 원래 잘하던 분야의 성능이 매우 악화됨
ex) 7B 언어모델로 vlm 훈련을 하면 자연어 생성 능력이 87퍼센트 떨어짐



arxiv.org/abs/2…
시각적 이해에 대한 능력을 향상시키면서 NLP 능력들을 잃지 않는게 가능한가? ->
가능함(P-Tuning과 LoRA 비교에서 영감을 받음)
학습가능한 visual experts 라는 것을 추가하면 됨 (MoE같은 느낌)
파라미터수는 2배가 되지만 FLOP은 동일하게
쉽게 말해
-> 이미지 없이 prompt 들어왔을 때는 기존의 LLM 파라미터 사용
이미지가 함께 prompt 들어왔을 때는 vlm 훈련된 파라미터 사용
2. Architecture
2-1. vision encoder - EVA2-CLIP-E 사용
([CLS] features 을 얻기 위한 contrasive learning을 위해 마지막 layer 제거)
2-2. MLP adapter -
ViT 결과를 text embedding 에 결합하기 위해 two-layer MLP swiglu 사용


2-3. pretrained language model
gpt style llm 전부 다 양립가능 , Vicuna1.5-7B를 further 훈련을 위해 사용
2-4. visual expert module
각 layer에 visual expert module 추가
visual expert module은 QKV 행렬, 각 layer의 MLP 를 갖고 있음
어텐션 layer : input shape
(배치사이즈, attention 헤드 수, (이미지 시퀸스 길이 + 텍스트 시퀸스 길이, hidden size)
밑의 과정을 통해
FFN i 버젼은 image , visual expert의 FFN이고
FFN t 버젼은 text, 원래 언어모델의 FFN

pretraining
LAION-2B and COYO-700M 데이터셋 , 손수 만든 데이터셋 사용
전처리 과정
LAION 2B COYO-700M
NSFW 내용, 깨진 url, 정치성향, 가로세로비 > 6 or < 1/6 제거 등의 전처리과정 거침
만든 거 - 4000만개
caption의 명사들은 이미지의 bounding box와 연관이 있는 명사
spaCy로 명사 추출하고
GLIPv2로 bounding box 예측하고
LAION-400M을 필터링한 -> LAION-115M으로 text- image 쌍 샘플링함
75퍼센트의 이미지가 적어도 2개의 bounding box를 가지고 있는 것을 확실시함

alignment

