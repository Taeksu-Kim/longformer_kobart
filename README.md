# longformer_kobart
transformers==4.3.3  
kobart encoder의 attention layer를 longfomer의 attention layer로 대체하여 input길이를 4096까지 늘리는 코드.

run test에서는 summarization으로 파인튜닝된 모델 사용

### 참고 소스    
https://github.com/allenai/longformer
https://huggingface.co/transformers/v4.3.3/_modules/transformers/models/bart/modeling_bart.html#BartModel


### 작업 내용
-  기존 Facebook bart 모델의 경우 기본 입력 최대길이가 1024이므로 1024*4로 4096의 max_pos값을 같지만, kobart는 길이가 1026이므로 값을 4104로 설정. 입력 길이는 홀수 값이 될 수 없는 attention window의 배수여야 하므로 facebook모델과 같은 4096 사용.
-  transformers==4.3.3를 사용하는 kobart에 맞게 버전에 맞는 LongformerAttention를 사용하기 위해 LongformerSelfAttentionForBart 수정. 
    -  attention_mask 차원 조정 bs x seq_len x seq_len -> bs x seq_len
    -  구 버전에는 LongformerAttention에 포함되어 있다가 바깥으로 나온 코드 추가, forward input 값들 수정

-  모델 저장 후 from_pretrained로 불러올 경우 config값이 모델 빌드에 제대로 적용되지 않아 LongformerEncoderDecoderForConditionalGeneration에서 모델 빌드 후 임베딩 레이어의 차원 재정의

- kobart에 맡게 BartTokenizer -> PreTrainedTokenizerFast로 토크나이저 수정

### Hyperparameter

max_pos: 모델에서 사용할 최대 input 길이. 기존 길이의 배수로 설정. facebook model은 1024, kobart는 1026.  
attention window: CNN의 컨볼루션과 비슷한 윈도우 사이즈. 홀수 값을 사용할 수 없음.  
max_seq_len: 실제 모델에 입력하는 데이터의 길이. longformer에서는 이 길이를 attention window의 배수로 맞춰줘야함.  

facebook 모델의 경우 1024 * 4, 512 * 8로 4096값을 사용할 수 있지만  
kobart 모델의 경우 1026 * 4, 512 * 8로 각각 4104, 4096의 값을 사용.
