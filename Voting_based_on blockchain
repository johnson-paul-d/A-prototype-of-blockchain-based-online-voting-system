import datetime
import hashlib
import json
import pandas as pd
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
from flask import Flask, jsonify, request, render_template, redirect, url_for, flash, session
import openpyxl

# Load the Excel file
wb = openpyxl.load_workbook('C:\\Users\\Admin\\OneDrive\\Desktop\\Reinforcement project\\voters_ids.xlsx')
sheet = wb.active

# Convert the Excel data to a pandas DataFrame
voter_df = pd.DataFrame(sheet.values)

app = Flask(__name__)
app.secret_key = '1234'  # Replace with a strong secret key


class Blockchain:
    def __init__(self):
        self.chain = []
        self.votes = {}  # To store the votes
        self.create_block(proof=1, previous_hash='0')

    def create_block(self, proof, previous_hash):
        block = {'index': len(self.chain) + 1,
                'timestamp': str(datetime.datetime.now()),
                'proof': proof,
                'previous_hash': previous_hash}
        self.chain.append(block)
        return block

    def proof_of_work(self, previous_proof):
        new_proof = 1
        check_proof = False
        while check_proof is False:
            hash_operation = hashlib.sha256(str(new_proof**2 - previous_proof**2).encode()).hexdigest()
            if hash_operation[:5] == '00000':
                check_proof = True
            else:
                new_proof += 1
        return new_proof

    def hash(self, block):
        encoded_block = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(encoded_block).hexdigest()

    def chain_valid(self, chain):
        previous_block = chain[0]
        block_index = 1
        while block_index < len(chain):
            block = chain[block_index]
            if block['previous_hash'] != self.hash(previous_block):
                return False
            previous_proof = previous_block['proof']
            proof = block['proof']
            hash_operation = hashlib.sha256(str(proof**2 - previous_proof**2).encode()).hexdigest()
            if hash_operation[:5] != '00000':
                return False
            previous_block = block
            block_index += 1
        return True

    def print_previous_block(self):
        return self.chain[-1]
    
def mine_block():
        previous_block = blockchain.chain[-1]
        previous_proof = previous_block['proof']
        proof = blockchain.proof_of_work(previous_proof)
        previous_hash = blockchain.hash(previous_block)
        block = blockchain.create_block(proof, previous_hash)

    
# Create the blockchain instance
blockchain = Blockchain()


# Define candidates
candidates = ["Andrew Tate","Elon Musk","Mark Zuckerberg"]


# Load voter IDs from Excel
voter_df = pd.read_excel('C:\\Users\\Admin\\OneDrive\\Desktop\\Reinforcement project\\voters_ids.xlsx')
voted_voters = set()  # Set to keep track of voters who have already voted

# Initialize blockchain
blockchain = Blockchain()

# Helper function to check voter ID
def is_valid_voter_id(voter_id):
    voter_df.columns = voter_df.columns.str.strip()
    voter_df['VoterID'] = voter_df['VoterID'].astype(str).str.strip()
    return voter_id in voter_df['VoterID'].values

@app.route('/')
def home():
    return render_template('welcome.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        try:
            voter_id = request.form.get('voter_id', '').strip()
            print(f"Entered Voter ID: '{voter_id}'")  # Debug print

            if is_valid_voter_id(voter_id):
                if voter_id not in voted_voters:
                    # Redirect to vote route with voter_id as URL parameter
                    return redirect(url_for('vote', voter_id=voter_id))
                else:
                    return "You have already voted!"
            else:
                return "Invalid Voter ID!"
        except Exception as e:
            print(f"An error occurred: {e}")
            return "An error occurred, please try again later."
    return render_template('login.html')

@app.route('/vote', methods=['GET', 'POST'])
def vote():
        voter_id = request.args.get('voter_id', '').strip()  # Get voter_id from URL parameters
        
        if request.method == 'POST':
            candidate = request.form.get('candidate', '').strip()

            # Check if the voter has already voted
            if voter_id in voted_voters:
                return "You have already voted!", 403

            # Validate candidate
            if candidate not in candidates:
                return "Invalid candidate!", 400

            # Record the vote
            blockchain.votes[candidate] = blockchain.votes.get(candidate, 0) + 1
            voted_voters.add(voter_id)

            # Mine a block to add the vote to the blockchain
            mine_block()

            # Flash a success message and redirect to home
            return("Your vote was recorded successfully!", "success")
        
        
        # Handle GET requests by rendering the voting page
        return render_template('vote.html', candidates=candidates)


def mine_block():
    previous_block = blockchain.chain[-1]
    previous_proof = previous_block['proof']
    proof = blockchain.proof_of_work(previous_proof)
    previous_hash = blockchain.hash(previous_block)
    blockchain.create_block(proof, previous_hash)

@app.route('/mine_block', methods=['GET'])
def mine_block_route():
    mine_block()
    block = blockchain.chain[-1]
    response = {
        'message': 'A block is MINED',
        'index': block['index'],
        'timestamp': block['timestamp'],
        'proof': block['proof'],
        'previous_hash': block['previous_hash']
    }
    return jsonify(response), 200

@app.route('/results')
def results():
    # Generate the bar chart
    plt.figure(figsize=(10, 6))
    plt.bar(blockchain.votes.keys(), blockchain.votes.values())
    plt.xlabel('Candidates')
    plt.ylabel('Votes')
    plt.title('Election Results')
    chart_path = "C:\\Users\\Admin\\OneDrive\\Desktop\\Reinforcement project\\static\\results.png"
    plt.savefig(chart_path)
    plt.close()  # Close the figure to free up memory

    # Prepare data for the table
    candidates = list(blockchain.votes.keys())
    votes = list(blockchain.votes.values())
    
    # Pass data to the template
    return render_template('results.html', candidates=candidates, votes=votes, chart_path=chart_path)


voting_open = True  # Global variable to track voting status

@app.route('/close_voting', methods=['GET'])
def close_voting():
    global voting_open
    voting_open = False
    return "Voting is now closed.", 200

@app.route('/view_blockchain', methods=['GET'])
def view_blockchain():
    print("Blockchain chain:", blockchain.chain)  # Debug print
    response = {
        'chain': blockchain.chain,
        'length': len(blockchain.chain)
    }
    return jsonify(response), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)