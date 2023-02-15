## Folder structure

```js
// Controllers with error handling for requests
controllers 
    -roomController.js
    -participantsController.js
// Connection to the database
lib 
    -connect_db.js
// Middleware for validation and reqest information
middleware 
    -validate.js
// Mongoose models and schema
models 
    -room.js
    -participant.js
// Routes including validation and controllers
routes 
    -schema.js
    -routes.js
.env
.env_sample
.gitignore
package.json
seed.js // Filling the db with mock data
server.js // Express and server configuration
```

## Server.js
```js
import express from "express";
import breakoutrooms from "./routes/routes.js";
import "./lib/connect_db.js";
import dotenv from "dotenv";
dotenv.config();


const app = express();
const port = process.env.PORT || 4000;

app.use(express.json()); // body parser enables access to req.body

app.use("/", breakoutrooms);

app.use((err, req, res, next) => {
    console.log(err);
    const statusCode = err.statusCode || 500;
    res.status(statusCode).send(err.message);
});

app.listen(port, () => console.log("Server is running at port: " + port));
```

## .env.sample

```js
PORT=4000
MONGODB_URI=
DATABASE="breakoutrooms"
```

## lib/connect_db.js
```js
import mongoose from "mongoose";
import dotenv from "dotenv";
dotenv.config();


mongoose.set("strictQuery", false);
mongoose
    .connect(process.env.MONGODB_URI, {
        dbName: process.env.DATABASE,
    })
    .then(() => console.log("connected via mongoose"));
```

## routes/routes.js
```js
import { Router } from "express";
import * as roomController from "../controllers/breakoutroomsController.js";
import * as participantController from "../controllers/participantsController.js";
import validate from "../middleware/validate.js";
import { schemaPostRooms } from "./breakoutroomsSchema.js";

const router = Router();

router
    .post("/rooms", validate(schemaPostRooms), roomController.postRoom)
    .post("/rooms/:roomId/participants/:participantId", roomController.postParticipant)
    .get("/rooms/", roomController.getAllRooms)
    .get("/rooms/:roomId", roomController.getRoomId)
    .get("/rooms/:roomId/participants", roomController.getRoomParticipants)
    .get("/rooms/:roomId/participants/email", roomController.getRoomParticipantsEmail)
    .get("/participants", participantController.getAllParticipants)
    
export default router;
```
## controllers/breakoutroomsController.js

```js
import * as Room from '../models/Breakoutroom.js';
import { faker } from '@faker-js/faker';

const errorSwitch = (err) => {
    switch(err.path) {
        case "_id":
            err.statusCode = 404
            err.message = "ID nicht gefunden"
            break
        default:
            err.statusCode = 400
            err.message = "Überprüfe deine Eingabe"
    }
    return err
}


export const postRoom = async (req, res, next) => {
    try {
        const result = await Room.create(req.body)
        res.status(201).json(result)
    } catch(err) {
        next(errorSwitch(err))
    }
}

export const postParticipant = async (req, res, next) => {
    try {
        console.log({room:req.params.roomId, participant: req.params.participantId});
        const result = await Room.postParticipant({participantId:req.params.participantId, roomId:req.params.roomId})
        res.status(200).json(result)
    } catch (err) {
        next(errorSwitch(err))
    }
}

export const getAllRooms = async (req, res, next) => {
    try {
        const result = await Room.getAllRooms()
        res.status(200).json(result)
    } catch (err) {
        next(errorSwitch(err))
    }
}

export const getRoomId = async (req, res, next) => {
    try {
        const result = await Room.getRoomId(req.params.roomId)
        res.status(200).json(result)
    } catch (err) {
        next(errorSwitch(err))
    }
}

export const getAllParticipants = async (req, res, next) => {
   
    try {
        const result = await Room.create(req.body)
        res.status(201).json(result)
    } catch(err) {
       next(errorSwitch(err))
    }
}

```
## models/Breakoutroom.js

```js
import mongoose from "mongoose";

const schema = new mongoose.Schema({
    number: {
        type: Number,
        required: true
    },
    date:{
        type:Date,
        default: Date.now
    },
    participants: [{
        type: mongoose.Schema.Types.ObjectId,
        ref: "Participant", // name of referenced model
    }]
})

const Breakoutroom = new mongoose.model("Breakoutroom", schema);

export const create = async ({number}) => {
    const newRoom = new Breakoutroom({number});
    const result = await newRoom.save();
    return result;
};

export const postParticipant = async ({participantId, roomId}) => {
    console.log({participantId, roomId});
    const result = await Breakoutroom.update({_id: roomId}, {$push:{participants:participantId}})
    return result;
};

export const getAllRooms = async () => {
    const result = await Breakoutroom.find().populate("participants", "name")
    return result;
};

export const getRoomId = async (roomId) => {
    const result = await Breakoutroom.findById(roomId).populate("participants", "name");
    return result;
};

export default Breakoutroom;
```

## middleware/validate.js

```js
import Ajv from "ajv";
import addFormats from "ajv-formats";
const ajv = new Ajv({allErrors: true});
addFormats(ajv);

const validate = (schema) => {
    const test = ajv.compile(schema);
    
    return (req, res, next) => {
        const valid = test(req.body);
        if (!valid) return res.status(400).json(test.errors)
        next();
    }
}

export default validate;
```

## routes/schema.js

```js
export const schemaPostRooms = {
    type: "object",
    properties: {
      number: {type: "number"},
      date: {type: "string", format: "date"},
      participants: {type: "string"},
    },
    required: ["number"],
    additionalProperties: false
  }
  
  export const getSChema = {
      type: "object",
      additionalProperties: false
  }
```

## seed.js

```js
import { faker } from "@faker-js/faker";
import Participant from "./models/Participant.js";
import Breakoutroom from "./models/Breakoutroom.js";
import "./lib/connect_db.js";


const deleteAllRooms = async() => {
    console.log("starting to delete");
    return await Participant.deleteMany();
};

const deleteAllParticipants = async() => {
    return await Breakoutroom.deleteMany();
};

const participants = [];
let counter = 1 // To get all participants into rooms

const createParticipant = async() => {
    const newParticipant = new Participant({
        name: faker.name.fullName(),
        city: faker.address.city(),
        email: faker.internet.email()
    })

    const result = await newParticipant.save();
    participants.push(result["_id"]);
}

const createRoom = async(num) => {
    const newRoom = new Breakoutroom({
        number: num,
        participants: getRandomIds()
    })

    const result = await newRoom.save();
}

const createParticipants =  async (count = 1) => {
    for (let i = 0; i < count; i++) {
        await createParticipant();
    }
}

const createRooms = async (count) => {
    for (let i = 0; i < count; i++) {
        await createRoom(i + 1);
    }
}

const getRandomIds = () => {
    if (participants.length < 1) return [];
    
    let max = participants.length;
    if (participants.length > 3) max = Math.abs(max/2);
    
    let number = Math.floor(Math.random()*(max - 1 + 1) + 1);
    const selection = [];
    
    if (counter === +process.argv[3]) number = participants.length;
    
    for (let i = 0; i < number; i++) {
        selection.push(participants[i]);
    }
    
    counter++;
    participants.splice(0, number);
    return selection;
}

try {
    if (!process.argv.includes("doNotDelete")) {
        console.log("Deleting all documents...");
        await deleteAllRooms();
        await deleteAllParticipants();
        console.log("finished deleting!");
    }

    const count1 = process.argv[2] === "doNotDelete" ? undefined : process.argv[2];
    const count2 = process.argv[3] === "doNotDelete" ? undefined : process.argv[3];

    await createParticipants(count1);
    console.log("Participants created!");
    await createRooms(count2);
    console.log("Rooms created!");

    process.exit(0);
} catch (error) {
    console.error(error);
    process.exit(1);
}
```

# AJV VALIDATION

## Folder structure

```js
middleware
    - validate.js
routes
    - reports.js
    - reports.schema.js
```

## middleware/validate.js

```js
import Ajv from "ajv";
const ajv = new Ajv({ allErrors: true });

const validate = (schema) => {
    const test = ajv.compile(schema);

    return (req, res, next) => {
        const valid = test(req.body);
        if (!valid) return res.status(400).json(test.errors);

        next();
    };
}

export default validate;
```

## routes/reports.schema.js

```js
export const postSchema = {
    type: "object",
    properties: {
        title: { type: "string" },
        description: { type: "string" },
        author: { type: "string" },
    },
    required: ["title"],
    additionalProperties: false,
};

export const getSchema = {
    type: "object",
    additionalProperties: false,
};

export const deleteSchema = {
    type: "object",
    additionalProperties: false,
};
```

## routes/reports.js

```js
import validate from "../middleware/validate.js";
import { postSchema, getSchema, deleteSchema } from "./reports.schema.js";

router.get("/", validate(getSchema), reports.getAll);
router.post("/", validate(postSchema), reports.create);
router.delete("/:id", validate(deleteSchema), reports.remove);
```