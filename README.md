# joi-type-extract
Clone of https://github.com/TCMiranda/joi-extract-type<br>

## Setup
1. Install package
	```bash
	npm install @goodrequest/joi-type-extract
	or
	yarn add @goodrequest/joi-type-extract
	```
2. Create ./src/types/joi/index.d.ts and import package in it, which will extend Joi types
   ```Typescript
   import '@goodrequest/joi-type-extract'
   ```
3. Add this compiler option and include created file in tsconfig.json in typeRoots
	```Json
	{
		"compilerOptions": {
            "strictNullChecks": true,
			"typeRoots": [
				"./src/types"
			]
		}
	}
	```

## Usage example

```Typescript
import Joi from 'joi'

export const requestSchema = () =>
	Joi.object({
		params: Joi.object({
			userID: Joi.number().required()
		}),
		query: Joi.object({
			search: Joi.string().required()
		}),
		body: Joi.object({
			name: Joi.string().required()
		})
	})

export const responseSchema = Joi.object({
   messages: Joi.array().items({
	   message: Joi.string().required(),
	   type: Joi.string().valid(MESSAGE_TYPE.SUCCESS).required()
   }).required()
})

type RequestType = Joi.extractType<ReturnType<typeof requestSchema>>
type ResponseType = Joi.extractType<typeof fullMessagesResponseSchema>
```
Extracted EequestType and ResponseType can be then used in express.Request and express.Response types
```Typescript
import { Request, Response, NextFunction } from 'express'

const businessLogic = async (
	req: Request<RequestType['params'], ResponseType, RequestType['body'], RequestType['query']>,
	res: Response<ResponseType>,
	next: NextFunction
) => {
	const { params, query, body } = req

	// params.userID will have number type
	// query.serch will have string type
	// body.name will have string type

	// response will be enforced to be same as ResponseType
	return res.json({
		messages: [
			{
				message: 'Response message',
				type: MESSAGE_TYPE.SUCCESS
			}
		]
	})
}
```
