<template>
  <div id="hello">
    <div class="container" style="width: 1500px">
				<div class="row" style="margin-top:50px">
					<div class="col-12" >
						<ul >
							<li class="col-2" v-for="product in productFire" :key="product['.key']">
									<a ><img :src="product.imgLink" width="300" height="255"></a><br/>
                    {{product.productName}}<br/>
  									<p class="price"><h5>{{product.price}} ฿</h5> </p>
                    <div v-if="product.amount !== 0 ">{{product.amount}} left</div>  <div v-if="product.amount === 0 " style="color:red">Out of Stock</div> <br>
                    <button type="button" class="btn btn-outline-primary" v-if="product.amount !==0" @click="addToCart(product.productName,product.price,product['.key'],product.amount,product.imgLink)">Add to cart</button>
							</li>
						</ul>
						<hr>
					</div>
          <div class="col-3" style="text-align: center" >
            <h2>Your Cart</h2>
             <div v-for="(item , index) in items">
               <ul>
                 <li class="list-group-item" style="width:350px; margin-bottom: 10px; " ><img :src="item.imgLink" width="50" height="50"> {{item.productName}} : {{item.price}} ฿ <br> {{item.amount}}<br>
                   <button class="btn btn-danger" style="margin-bottom: 20px; margin-top: 20px" @click="pickOff(item.productName,item.price,item.amount,item.imgLink,item.key,index,item.amountf)">Remove</button></li>
               </ul>
             </div>

            All of these : <strong>{{cost}} ฿ </strong><br>
         <button type="button" class="btn btn-outline-success" @click="checkOut">Check out</button>
                <!-- <button @click="removeTodoFire(todo['.key'])">X</button> -->
          </div>
				 </div>
        <div class="" v-if="inputPassword!=passwordForAdd" style="margin-bottom: 30px">
          <h2>Enter password to add product</h2>
          <input type="password" class="form-control" placeholder="Password" v-model="inputPassword" >
        </div>
      <div class="" v-if="inputPassword==passwordForAdd" >
        <div>
          <h1>Add product</h1> <br>
        </div>

        <div class="row" style="margin-bottom: 30px">

          <div class="col">
            <input type="text" class="form-control" placeholder="Name" v-model="n" >
          </div>
          <div class="col">
            <input type="number" class="form-control" min="1" placeholder="Price" v-model="p" >
          </div>
          <div class="col">
            <input type="number" class="form-control" min="1" placeholder="Amount" v-model="a" >
          </div>
          <div class="col">
            <input type="text" class="form-control" placeholder="Image link" v-model="l" >
          </div>
          <button class="btn btn-success" @click="addProduct"> Add </button>
          <button class="btn btn-danger" @click="changePassword"> Cancel </button>
        </div>
      </div>
		</div>
		</div>
    <!-- firebase -->
    <!-- <ul>
      <li v-for="todo in todosFire" :key="todo['.key']">
        <input type="text" :value="todo.text" @input="updateTodoFire(todo['.key'], $event.target.value)">
        <button @click="removeTodoFire(todo['.key'])">X</button>
      </li>
    </ul> -->
</template>
<script>
import firebase from 'firebase'
import swal from 'sweetalert2'
var config = {
    apiKey: "AIzaSyB70OjYu4V1XLc5wnVnG3s_Wu0Bh2hhhsg",
    authDomain: "advweb-be43b.firebaseapp.com",
    databaseURL: "https://advweb-be43b.firebaseio.com",
    projectId: "advweb-be43b",
    storageBucket: "",
    messagingSenderId: "731624869611"
  }
var firebaseApp = firebase.initializeApp(config)
var db = firebaseApp.database()
var todosRef = db.ref('todos')
var productRef = db.ref('products')
export default {
  name: 'HelloWorld',
  firebase: {
    todosFire: todosRef,
    productFire: productRef
  },
  data () {
    return {
      text: '',
      productName: '',
      price: '',
      addProductButtonState: false,
      inputPassword: '',
      passwordForAdd: '123456',
      authorized: false,
      onAdd: false,
      p: null,
      n: null,
      l: null,
      a: null,
      cost: 0,
      startA: 1,
      items: [],
      localAmout : 0
    }
  },
  methods: {
    changePassword () {
      this.inputPassword = ''
    },
    pickOff (name,price,amount,link,key,index,amountf) {
      console.log(amountf)
      db.ref('products/' + key).set({
        productName: name,
        price: price,
        imgLink: link,
        amount: amountf+amount-1
      })
      this.cost-=price * amount
      this.items.splice(index, 1)
    },
    checkOut () {
       swal({
        title: 'คุณแน่ใจที่จะซื้อสิ้นค่าใช่หรือไม่?',
        text: "เราจะไม่รับเงินคืน",
        type: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'ใช่',
        cancelButtonText: 'ไม่ใช่',
      }).then((result) => {
      if (result.value) {
        swal(
          'สั่งซื้อสินค้าเรียบร้อย',
          'ขอบคุณที่ไว้ใจเรา',
          'success'
        )
       this.items = []
       this.cost = 0
        }
    })

    },
    addToCart (name,price,key,amount,link) {
      console.log(amount)
      if (this.items[0] == null) {
        this.cost += price * 1
        this.items.push({
          productName: name,
          price: price,
          key: key,
          amount: 1,
          imgLink: link,
          amountf: amount})

          db.ref('products/' + key).set({
            productName: name,
            price: price,
            imgLink: link,
            amount: amount - 1
          })
      } else {
        console.log("we're inside else")
        for (var i = 0; i < this.items.length; i++) {
          if (name === this.items[i].productName) {
            console.log("we're inside if")
            this.cost += price * 1
            db.ref('products/' + key).set({
              productName: name,
              price: price,
              imgLink: link,
              amount: amount - 1
            })
            this.items[i].productName = name
            this.items[i].price = price
            this.items[i].key = key
            this.items[i].amount = this.items[i].amount += 1
            this.items[i].imgLink = link
            this.items[i].amountf = amount
            // this.items[i].push({
            //   productName: name,
            //   price: price,
            //   key: key,
            //   amount: this.items[i].amount += 1,
            //   imgLink: link,
            //   amountf: amount})


          } else if (name !== this.items[i].productName) {
            this.cost += price * 1
            db.ref('products/' + key).set({
              productName: name,
              price: price,
              imgLink: link,
              amount: amount - 1
            })
            this.items.push({
              productName: name,
              price: price,
              key: key,
              amount: 1,
              imgLink: link,
              amountf: amount})
              break
          }
        }
      }
    },
    addProduct () {
      if (this.n!==null||this.p!==null||this.a!==null||this.l!==null) {
        if (this.n!==null) {
          if (this.p!==null&&this.p>=0) {
            if (this.a!==null&&this.a>0) {
              if (this.l!==null) {
                productRef.push({
                  productName: this.n,
                  price: this.p,
                  imgLink: this.l,
                  amount: this.a
                })
                this.n = null
                this.l = null
                this.p = null
                this.a = null
              }else {
                swal(
                  'Warning!',
                  'Please fill in image\'s link of product!',
                  'warning'
                  )
              }
            }else {
              swal(
                'Warning!',
                'Please fill in the correct amount of product!',
                'warning'
                )
            }
          }else {
            swal(
              'Warning!',
              'Please fill in the correct price of product!',
              'warning'
              )
          }
        }else {
          swal(
            'Warning!',
            'Please fill in name of product!',
            'warning'
            )
        }
      }else {
        swal(
          'Warning!',
          'Please fill up the information!',
          'warning'
          )
      }
    },
    addTodoFire () {
      todosRef.push({
        text: 'wsdsd',
        productName: 'Coca col',
        price: 125

      })
    },
    updateTodoFire (key, text) {
      // todosRef.child(key).child('text').set(text)
      db.ref('todos/' + key).set({
        text: text
      })
    },
    removeTodoFire (key) {
      // todosRef.child(key).remove()
      db.ref('todos/' + key).remove()
    },
    changeStateButton () {
      this.addProductButtonState = true
    },
    changeStatePassword () {
      this.onAdd = true
    },
    updateTodo (key, text) {
      // const index = this.todos.findIndex((todo) => todo.key === key)
      // if (index > -1) {
      //   this.todos[index].text = text
      // }
      const todo = this.todos.find((todo) => todo.key === key)
      if (todo) {
        todo.text = text
      }
    },
    removeTodo (key) {
      // const index = this.todos.findIndex(function (todo) {
      //   return todo.key === key
      // })
      const index = this.todos.findIndex((todo) => todo.key === key)
      if (index > -1) {
        this.todos.splice(index, 1)
      }
    }
  }
}
</script>

<style>
  #app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: -5px;
  }
  ul {
    margin: 0 auto;
    text-align: center;
  }

  li {
    display: inline-block;
    vertical-align: top;
    margin-bottom: 10px
  }
  .imgEdit {
    border-radius: 50%;
    width: 300px;
    height: 300px;
  }
  h1 {
     font-size: 50px;
     color: black;
     text-align: center;
     text-shadow: 0px 1px 0px #f2f2f2;
     border: 1px solid transparent;
   }
  h2 {
     font-size: 35px;
     color: black;
     text-align: center;
     margin: 0 0 35px 0;
     text-shadow: 0px 1px 0px #f2f2f2;
   }
 h5 {
      font-size: 20px;
      color: black;
      text-align: center;
      font-weight: bold;
      text-shadow: 0px 1px 0px #f2f2f2;
    }
  .borderlist {
    list-style-position:inside;
    border: 1px solid black;
  }
</style>
