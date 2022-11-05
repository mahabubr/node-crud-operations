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

        app.get('/users/:id', async (req, res) => {
            const id = req.params.id
            const query = { _id: ObjectId(id) }
            const user = await userCollection.findOne(query)
            res.send(user)
        })

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
