<template>
  <div class = "userReviewItem">
    <div class = "reviewContext"> 
      <div class = "reviewItemContent">
        <form @submit.prevent="handleSubmit(review.id)">
            Are you sure you want to delete this review?
            <br>
            <br>
            <button v-on:click="state.confirm = false" class="button">Nah, nevermind</button> &emsp;
            <button class="button">Confirm deletion</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, onMounted } from "vue";
import { useStore } from 'vuex';

export default {
    name: "Edit",
    setup(props, ctx) {
        // make sure accesssing outside of substring doesn't break it. Doesn't seem to right now.
        // Adds functionality for expanding reviews
        const store = useStore();
        const state = reactive({
            edit: false,
            username:  '',
            review:  [],
            confirm: true
        });


        onMounted(() => {
          state.username = props.username;
          props.edit == false;
          state.review = props.review
        })

      function handleSubmit(id) {
      // scuffed because I can't call await methods without it being in a const function
      // converts review id into int, is string. Need to figure out why this happens but for now I want it fixed.
        //state.review.id = parseInt(state.review.id)
        if(state.confirm == true) {
            const data = [props.review.id, state.username]
            console.log("data")
            console.log(data)
            deleteReview(data);
        }
        else {
            resetUser(state.username)
        }
        ctx.emit("showOptions", id);

      }

      function deleteReview(id) {
      // scuffed because I can't call await methods without it being in a const function
        const data = [id, state.username]

        sendDelete(data);
      }

      const sendDelete = async (data) => {
        await fetch('http://3.133.58.37:5000/delete', {
        method: 'POST',
        body: JSON.stringify({ data }),
        headers: {
            'Content-type': 'application/json',
        }
        })
        .then((response) => response.json())        // flask returns a response object
        .then(function (user) {
            console.log(user);        // error catch is based on response. Not sure if works --> Also also needs>
            setUser(user)
        })
        .catch(function (error) {
          console.warn('Something went horribly wrong -->', error);
        });
      }

        const resetUser = async (userId) => {
            await fetch('http://3.133.58.37:5000/home', {
            method: 'POST',
            body: JSON.stringify({ userId }),
            headers: {
                'Content-type': 'application/json',
            }
            })
            .then((response) => response.json())        // flask returns a response object
            .then(function (user) {
                console.log("user")
                console.log(user);        // error catch is based on response. Not sure if works --> Also also needs>
                setUser(user)
            })
            .catch(function (error) {
            console.warn('Something went horribly wrong -->', error);
            });
        }

      // for updating state-user according to database
      const setUser = async (user) => {
        await store.dispatch('User/setUser', user);
        console.log(user.Item.username)
      }

        return {
            handleSubmit,
            deleteReview,
            setUser,
            sendDelete,
            resetUser,
            state
        };
    },
    props: {
        username: {
            type: String,
            required: true
        },
        review: {
            type: Object,
            required: true
        },
        edit: {
            type: Boolean,
            required: true
        }
    },
};
</script>

<style>
.reviewItem {
    padding: 12px;
    background-color: #ececec;
    border-radius: 5px;
    border: 1px solid #0d1424;
    box-sizing: border-box;
    cursor: pointer;
    transition: all 0.24s ease;
    width: 550px;
    margin: auto;
    margin-top: 5px;
    font-size: 18px;   

}

    .userReviewItem {
        font-weight: bold;
    }

    .reviewContext {
        font-weight: normal;
    }

    input[type=text] {
      background-color: #ececec;
      font-family: Avenir, Helvetica, Arial, sans-serif;
      color: #0d1424;
      border: none;
      font-size: 16px;  
    }

    textarea {
      background-color: #ececec;
      font-family: Avenir, Helvetica, Arial, sans-serif;
      color: #0d1424;
      border: none;
      font-size: 18px;  
      resize: none;
      text-align: center;
    }

    input[type=text]:focus {
      background-color: #ececec;
      font-family: Avenir, Helvetica, Arial, sans-serif;
      color: #0d1424;
      border: none;
      font-size: 18px;    
    }

    text {
      font-size: 16px;  
      text-align: center; 
    }
</style>