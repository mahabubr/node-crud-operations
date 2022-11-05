# Form OnBlue

```
const [user, setUser] = useState({})

const handleInputBlur = (event) => {
    const value = event.target.value
    const field = event.target.name
    const newUser = { ...user }
    newUser[field] = value
    setUser(newUser)
}
```

```
<form onSubmit={handleAddUser}>
    <input onBlur={handleInputBlur} type="text" name="name" placeholder='Name' />
    <br />
    <input onBlur={handleInputBlur} type="text" name="address" placeholder='Address' />
    <br />
    <input onBlur={handleInputBlur} type="email" name="email" placeholder='Email' />
    <br />
    <button type="submit">Add User</button>
</form>
```
<br/>
<br/>

# Create Method

## Client Side

```
fetch('http://localhost:5000/users', {
            method: "POST",
            headers: {
                "content-type": "application/json"
            },
            body: JSON.stringify(user)
        })
            .then(res => res.json())
            .then(data => {
                if (data.acknowledged) {
                    alert('user added successfully')
                    event.target.reset()
                }
                else {
                    alert('something wrong')
                }
            })
```

<br/>

## Server Side

```
async function run() {
    try {
        const userCollection = client.db('nodeMongoCrud').collection('users')

        app.post('/users', async (req, res) => {
            const user = req.body
            const result = await userCollection.insertOne(user)
            res.send(result)
        })

    }
    catch(e) {
        console.error(e)
    }
}

run()
```

<br/>
<br/>

# Read Method

## Client Side

```
fetch('http://localhost:5000/users')
        .then(res => res.json())
        .then(data => setUser(data))
```

<br/>

## Server Side

```
async function run() {
    try {
        const userCollection = client.db('nodeMongoCrud').collection('users')

        app.get('/users', async (req, res) => {
            const query = {}
            const cursor = userCollection.find(query)
            const users = await cursor.toArray()
            res.send(users)
        })

        app.get('/services/:id', async (req, res) => {
            const id = req.params.id
            const query = { _id: ObjectId(id) }
            const service = await serviceCollection.findOne(query)
            res.send(service)
        })

    }
    catch(e) {
        console.error(e)
    }
}

run()
```

<br/>
<br/>

# Update Method

## Client Side

```
{
    path: '/update/:id',
    element: <Update></Update>,
    loader: async ({ params }) => fetch(`http://localhost:5000/users/${params.id}`)
}

const storeUser = useLoaderData()
const [user, setUser] = useState(storeUser)

<Link to={`/update/${user._id}`}><button>Update</button></Link>
```

```
fetch(`http://localhost:5000/users/${storeUser._id}`, {
            method: "PUT/PATCH",
            headers: {
                "content-type": "application/json"
            },
            body: JSON.stringify(user)
        })
            .then(res => res.json())
            .then(data => {
                if (data.modifiedCount > 0) {
                    alert('user updated')
                }
            })
```

<br/>

## Server Side

```
async function run() {
    try {
        const userCollection = client.db('nodeMongoCrud').collection('users')

        app.put('/users/:id', async (req, res) => {
            const id = req.params.id
            const filter = { _id: ObjectId(id) }
            const user = req.body
            const option = { upsert: true }
            const updatedUser = {
                $set: {
                    name: user.name,
                    address: user.address,
                    email: user.email
                }
            }
            const result = await userCollection.updateOne(filter, updatedUser, option)
            res.send(result)
        })

    }
    catch(e) {
        console.error(e)
    }
}

run()
```

<br/>

## Alternative PATCH

### Client Side

```
<button onClick={() => handleStatusUpdate(_id)}className="btn btn-ghost btn-xs">{status ? status : 'pending'}</button>
```
```
 const handleStatusUpdate = id => {
        fetch(`http://localhost:5000/orders/${id}`, {
            method: 'PATCH', 
            headers: {
                'content-type': 'application/json'
            },
            body: JSON.stringify({status: 'Approved'})
        })
        .then(res => res.json())
        .then(data => {
            console.log(data);
            if(data.modifiedCount > 0) {
                const remaining = orders.filter(odr => odr._id !== id);
                const approving = orders.find(odr => odr._id === id);
                approving.status = 'Approved'

                const newOrders = [approving, ...remaining];
                setOrders(newOrders);
            }
        })
    }
```

### Server Side
```
async function run() {
    try {
        const userCollection = client.db('nodeMongoCrud').collection('users')

        app.patch('/orders/:id', async (req, res) => {
        const id = req.params.id;
        const status = req.body.status
        const query = { _id: ObjectId(id) }
        const updatedDoc = {
            $set:{
                status: status
            }
        }
        const result = await orderCollection.updateOne(query, updatedDoc);
        res.send(result);
    })
}
catch(e) {
    console.error(e)
    }
}

run()
```

<br/>
<br/>

# Delete Method

## Client Side

```
<button onClick={() => handleDelete(user)}>X</button>
const [displayUsers, setDisplayUsers] = useState(users)
```

```
const handleDelete = (user) => {

        const agree = window.confirm(`are you sure want to delete user ${user.name}`)

        if (agree) {
            fetch(`http://localhost:5000/users/${user._id}`, {
                method: "DELETE"
            })
                .then(res => res.json())
                .then(data => {
                    if (data.deletedCount > 0) {
                        alert('deleted successfully')
                        const remainingUsers = displayUsers.filter(usr => usr._id !== user._id)
                        setDisplayUsers(remainingUsers)
                    }
                })
        }
    }
```

<br/>

## Server Side

```
async function run() {
    try {
        const userCollection = client.db('nodeMongoCrud').collection('users')

        app.delete('/users/:id', async (req, res) => {
            const id = req.params.id
            const query = { _id: ObjectId(id) }
            const result = await userCollection.deleteOne(query)
            res.send(result)
        })

    }
    catch(e) {
        console.error(e)
    }
}

run()
```

<br/>
<br/>
<br/>

# Query Parameter Load Data

## Client Side

```
const [orders, setOrders] = useState([])

useEffect(() => {
    fetch(`http://localhost:5000/orders?email=${user?.email}`)
        .then(res => res.json())
        .then(data => setOrders(data))
}, [user?.email])
```

<br/>

## Server Side

```
async function run() {
    try {

    const userCollection = client.db('nodeMongoCrud').collection('users')

    app.get('/orders', async (req, res) => {

    let query = {}
    if (req.query.email) {
        query = {
            email: req.query.email
        }
    }
    const cursor = orderCollection.find(query)
    const orders = await cursor.toArray()
    res.send(orders)
})

}
catch(e) {
    console.error(e)
    }
}

run()
```

<br/>
<br/>
<br/>

# Pagination

## Client Side

```
// const { products, count } = useLoaderData();

const [products, setProducts] = useState([]);
const [count, setCount] = useState(0);
const [cart, setCart] = useState([]);
const [page, setPage] = useState(0);
const [size, setSize] = useState(10);
```

```
const pages = Math.ceil(count / size);
```

```
{
    [...Array(pages).keys()].map(number => 
<button
    key={number}
    className={page === number ? 'selected' : ''}
    onClick={() => setPage(number)}
    >
    {number + 1}
</button>)
}
```

```
<select onChange={event => setSize(event.target.value)}>
    <option value="5">5</option>
    <option value="10" selected>10</option>
    <option value="15">15</option>
    <option value="20">20</option>
</select>
```

```
useEffect(() => {
    const url = `http://localhost:5000/products?page=${page}&size=${size}`;
    fetch(url)
        .then(res => res.json())
        .then(data => {
            setCount(data.count);
            setProducts(data.products);
        })
}, [page, size])
```
<br/>

## Server Side

```
async function run() {
    try {

    const userCollection = client.db('nodeMongoCrud').collection('users')

    app.get('/products', async(req, res) =>{
        const page = parseInt(req.query.page);
        const size = parseInt(req.query.size);
        const query = {}
        const cursor = productCollection.find(query);
        const products = await cursor.skip(page*size).limit(size).toArray();
        const count = await productCollection.estimatedDocumentCount();
        res.send({count, products});
    });

}
catch(e) {
    console.error(e)
    }
}

run()
```

<br>
<br>

# Local Storage Connect Post

## Client Side

```
useEffect(() => {
        const storedCart = getStoredCart();
        const savedCart = [];
        const ids = Object.keys(storedCart); 
        console.log(ids);
        fetch('http://localhost:5000/productsByIds', {
            method: 'POST',
            headers: {
                'content-type': 'application/json'
            },
            body: JSON.stringify(ids)
        })
        .then(res => res.json())
        .then(data => {
            for (const id in storedCart) {
            const addedProduct = data.find(product => product._id === id);
            if (addedProduct) {
                const quantity = storedCart[id];
                addedProduct.quantity = quantity;
                savedCart.push(addedProduct);
            }
        }
        setCart(savedCart);

        })
        
    }, [products])
```
<br />

## Server Side

```
async function run() {
    try {

    const userCollection = client.db('nodeMongoCrud').collection('users')

    app.post('/productsByIds', async(req, res) =>{
        const ids = req.body;
        const objectIds = ids.map(id => ObjectId(id))
        const query = {_id: {$in: objectIds}};
        const cursor = productCollection.find(query);
        const products = await cursor.toArray();
        res.send(products);
    })

}
catch(e) {
    console.error(e)
    }
}

run()
```
