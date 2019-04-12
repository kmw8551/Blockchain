### Blockchain Practice 

강의자료를 참고한 블록체인 작업증명 및 동작 코드 구현
Blockchain A-Z™: Learn How To Build Your First Blockchain -> 참고 강의

``` python


#Postman HTTP Client:
import datetime
import hashlib
import json
from flask import Flask, jsonify

#Building a Blockchain
class  Blockchain:
    def __init__(self):
        #initialize chain
        self.chain = []
        #genesis block, so proof =1 previous_hash should be 0
        self.create_block(proof = 1, previous_hash = '0')
        
    def create_block(self, proof, previous_hash):
        #block을 dictionary 형태로 만들기
        block = {"index" : len(self.chain)+1,
                 "timestamp" : str(datetime.datetime.now()),  
                 #timestamp 문자열로 한것은 JSON 포맷에 나중에 변환하기 위함
                 "proof" : proof,
                 "previous_hash" : previous_hash}
        self.chain.append(block)
        return block
    
    def get_previous_block(self):
        #리스트의 마지막 것만 전달해주면 됨
        return self.chain[-1]

    #proof of work 만들기
    def proof_of_work(self, previous_proof):
        new_proof = 1 
        #1부터 시작하며 while 루프문을 통해 반복해서 블록을 찾고 값을 하나 올리는 방식으로 접근        
        check_proof = False
        #Check_proof가 True가 될 때 while 계산이 멈춤
        
        while check_proof is False:
        #4 leading zeros =>실제로 쓰일 때, 여기 예제에선 안 쓰임    
            hash_operation = hashlib.sha256(str(new_proof**2 - previous_proof**2).encode()).hexdigest()
            #encode 함수는 암호화 한 것
            #new_proof - previous_proof 이유
            
            #만약 new_proof + previous_proof로 하면 대칭이 되버림
            #(previous_proof + new_proof)가 가능하게 된다는 소리,
            #이건 대칭 암호화, 관련해서 사용하지 않을 것.
            
            if hash_operation[:4] == "0000":
                check_proof = True
            else:
                new_proof += 1
        return new_proof
        # 만약 new_proof 를 사용한 계산이 계속 False 값이면
        # 바꿔줘가면서 해야 연산이 되고,  '0000'을 찾을 것임
        # 따라서 +=1로 new_proof를 업그레이드 시켜준다
        
    def hash(self, block):
        #make block to string
        #JSON format 변환을 위해서 
        encoded_block = json.dumps(block, sort_keys=True).encode()
        #JSON 문자열로 변환
        return hashlib.sha256(encoded_block).hexdigest() 
    
    def is_chain_valid(self, chain):
        #initialize block index, previous block
        previous_block = chain[0]
        block_index = 1
        while block_index < len(chain):
            
            block = chain[block_index]
            if block["previous_hash"] != self.hash(previous_block):
                return False
            previous_proof = previous_block['proof']
            proof = block['proof']
            #block의 proof 이어받아서 밑에 계산에 대입
            hash_operation = hashlib.sha256(str(proof**2 - previous_proof**2).encode()).hexdigest()
            #위에 new_proof가 아니라 proof여야한다. 조심
            if hash_operation[:4] != '0000':
                return False
            previous_block = block 
            #검증 통과후 이전 블록 공간에 새 블록을 할당
            #그렇게 체인에 블록 추가
            #이후 블록 인덱스값 올려주기
            block_index += 1
        return True
    
#flask http://flask.pocoo.org/docs/1.0/quickstart/
#Creating a Web app
app = Flask(__name__)

#Creating a Blockchain
blockchain = Blockchain()          
            
#Mining a new Block
@app.route('/mine_block', methods=['GET']) #아직 post는 필요없음
#get the proof , proof of work is required
def mine_block(): #인자값 필요 없음 이미 위에 클래스에 다 정의해놓음
    
    previous_block = blockchain.get_previous_block() 
    previous_proof = previous_block['proof']
    proof = blockchain.proof_of_work(previous_proof)
    previous_hash = blockchain.hash(previous_block)
    block = blockchain.create_block(proof, previous_hash)
    response = {'message': 'Congrats! you just mined a block!!',
                'index' : block['index'],
                'timestamp' : block['timestamp'],
                'proof' : block['proof'],
                'previous_hash' : block['previous_hash']}
    return jsonify(response), 200
    #http status code 반환 (200은 제대로 성공했다는 )

#Getting the full Blockchain
@app.route('/get_chain', methods=['GET'])
def get_chain():
    response = {'chain' : blockchain.chain,
                'length' : len(blockchain.chain)}
      
    return jsonify(response), 200       
            
@app.route('/is_valid', methods=['GET'])
def is_valid():
    is_valid = blockchain.is_chain_valid(blockchain.chain)
    if is_valid :
        response = {'message': 'All good. The Blockchain is valid.'}
    else:
        response ={'message': 'Invalid block, Review your code'}
    
    return jsonify(response), 200

           
#Running the app
app.run(host = '0.0.0.0', port = 5000)            
#플라스크는 포트번호 5000에서 작동            
            
            
            
            
            
            
            

```
