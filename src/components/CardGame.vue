<script>
import { ref } from 'vue';
import axios from "axios";
    function translateCards(value){
        switch (value) {
            case 'JACK':
                value = '11';
                break;
            case 'QUEEN':
                value = '12';
                break;
            case 'KING':
                value = '13';
                break;
            case 'ACE':
                value = '14';
                break;
            default:
                break;
        }
        return value;
    }
    export default {
        name: 'CardGame',
        setup(){
            let gameOver = ref(true),
                cardOne = ref({}),
                cardTwo = ref({}),
                deckId = ref(null),
                playerOneScore = ref(0),
                playerTwoScore = ref(0);

            async function getDeck() {
                gameOver.value = false;
                playerOneScore.value = 0;
                playerTwoScore.value = 0;

                if (deckId.value === null) {
                    const { data } = await axios.get('https://www.deckofcardsapi.com/api/deck/new/shuffle/?deck_count=1')
                    deckId.value = data.deck_id;
                }
             }

             

             async function getCards() {
                const { data } = await axios.get(
                    `https://www.deckofcardsapi.com/api/deck/${deckId.value}/draw/?count=2`
                );
                const cardsRemaining = data.remaining;
                const { cards } = data;
                cardOne.value = cards[0];
                cardTwo.value = cards[1];
                console.info('remaining', cardsRemaining);
                const valueOne = parseInt(translateCards(cardOne.value.value));
                const valueTwo = parseInt(translateCards(cardTwo.value.value));
                if(valueOne > valueTwo) playerOneScore.value += 1;
                if(valueOne < valueTwo) playerTwoScore.value += 1;

                if(cardsRemaining === 0){
                    gameOver.value = true;
                }
             }
             
             return {
                gameOver,
                cardOne,
                cardTwo,
                deckId,
                playerOneScore,
                playerTwoScore,
                getCards,
                getDeck
             }
        }
        
    }
</script>
<template>
   
        <div class="row" v-if="!gameOver && (cardOne.image !== undefined && cardTwo.image !== undefined)">
            <div class="col sm12">
                 <div id="playerOne">
                   
                    <div class="card">
                        <div class="card-content">
                            <div class="card-title">
                                <span>Player One</span>
                            </div>
                            <img :src="cardOne?.images?.png" alt="Card One">
                        </div>
                    </div>
                </div>
            </div>
            <div class="col sm12">
                 <div id="playerTwo">
                    <div class="card">
                        <div class="card-content">
                             <div class="card-title">
                                <span>Player Two</span>
                            </div>
                            <img :src="cardTwo?.images?.png" alt="Card Two">
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div id="gameOver" class="row col sm6" v-if="gameOver">
            <div class="card"  v-if="playerOneScore !== 0 || playerTwoScore !== 0">
                <div class="card-content">
                    <div class="title">
                        <h4>Player {{ playerOneScore > playerTwoScore ? 'One' : 'Two' }} Wins!</h4>
                    </div>
                    <i class="large material-icons">emoji_events</i>
                </div>
            </div>
            
            <button class="btn light-ripple" @click="getDeck()">Play Game</button>
        </div>
         <div id="scoreboard" class="col sm12" v-if="deckId !== null && !gameOver">
                <button @click="getCards()" class="btn light-ripple"> Draw Cards</button>
                <h4>Player One: {{ playerOneScore }} | Player Two: {{ playerTwoScore }}</h4>
        </div>

  
</template>
<style>

</style>