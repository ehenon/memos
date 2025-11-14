# OpenAPI discriminators in the context of a TypeScript NestJS HTTP API

Properly manage the concept of OpenAPI discriminators on a NestJS API HTTP endpoint in TypeScript:
- Generation of [clean OpenAPI specs](https://swagger.io/docs/specification/v3_0/data-models/inheritance-and-polymorphism/#discriminator) via the [`@nestjs/swagger`](https://github.com/nestjs/swagger) library
- Proper management of request/response types
- Proper validation of all `400 Bad Request` cases

## Controller side

```typescript
import { Body, Controller, HttpStatus, Post } from "@nestjs/common";
import { ApiOperation, ApiResponse } from "@nestjs/swagger";
import {
  DiscriminatorType,
  TestDiscriminatorsRequest,
  TestDiscriminatorsResponse,
} from "./items.dto";

@Controller()
export class ItemsController {
  constructor() {}

  @Post("/")
  @ApiOperation({ summary: "Endpoint for testing discriminators" })
  @ApiResponse({ status: HttpStatus.CREATED, type: TestDiscriminatorsResponse })
  @ApiResponse({ status: HttpStatus.BAD_REQUEST })
  async testDiscriminators(@Body() requestBody: TestDiscriminatorsRequest): Promise<TestDiscriminatorsResponse> {
    // Check request body after class-transformer
    console.log("Transformed request body = ", JSON.stringify(requestBody, null, 2));

    // Fake return to test response types and autocomplete
    return {
      data: {
        type: DiscriminatorType.A,
        data: {
          shapeA1stField: "toto",
          shapeA2ndField: true,
        },
      },
    };
  }
}
```

## DTOs side

```typescript
import { BadRequestException } from "@nestjs/common";
import { ApiExtraModels, ApiProperty, getSchemaPath } from "@nestjs/swagger";
import { Type } from "class-transformer";
import { IsBoolean, IsDefined, IsIn, IsString, ValidateNested } from "class-validator";

export enum DiscriminatorType {
  A = "A",
  B = "B",
}

class RequestDataShapeA {
  @ApiProperty({ type: String, description: "1st field of data shape A" })
  @IsString()
  shapeA1stField: string;

  @ApiProperty({ type: Boolean, description: "2nd field of data shape A" })
  @IsBoolean()
  shapeA2ndField: boolean;
}

class RequestADto {
  @ApiProperty({ enum: [DiscriminatorType.A], description: "Discriminator type A" })
  @IsIn([DiscriminatorType.A])
  type: DiscriminatorType.A;

  @ApiProperty({ type: RequestDataShapeA })
  @IsDefined()
  @ValidateNested()
  @Type(() => RequestDataShapeA)
  data: RequestDataShapeA;
}

class RequestDataShapeB {
  @ApiProperty({ type: String, description: "1st field of data shape B" })
  @IsString()
  shapeB1stField: string;

  @ApiProperty({ type: Boolean, description: "2nd field of data shape B" })
  @IsBoolean()
  shapeB2ndField: boolean;
}

class RequestBDto {
  @ApiProperty({ enum: [DiscriminatorType.B], description: "Discriminator type B" })
  @IsIn([DiscriminatorType.B])
  type: DiscriminatorType.B;

  @ApiProperty({ type: RequestDataShapeB })
  @IsDefined()
  @ValidateNested()
  @Type(() => RequestDataShapeB)
  data: RequestDataShapeB;
}

@ApiExtraModels(RequestADto, RequestBDto)
export class TestDiscriminatorsRequest {
  @ApiProperty({
    oneOf: [{ $ref: getSchemaPath(RequestADto) }, { $ref: getSchemaPath(RequestBDto) }],
    description: "Request data, which can be of shape A or shape B depending on the type",
    discriminator: {
      propertyName: "type",
      mapping: {
        [DiscriminatorType.A]: getSchemaPath(RequestADto),
        [DiscriminatorType.B]: getSchemaPath(RequestBDto),
      },
    },
  })
  @IsDefined()
  @ValidateNested()
  @Type(
    (opts) => {
      if (opts?.object.request.type === DiscriminatorType.A) return RequestADto;
      if (opts?.object.request.type === DiscriminatorType.B) return RequestBDto;
      throw new BadRequestException(
        `type must be one of the following values:${Object.values(DiscriminatorType).map((v) => ` ${v}`)}`,
      );
    },
    {
      discriminator: {
        property: "type",
        subTypes: [
          { name: "A", value: RequestADto },
          { name: "B", value: RequestBDto },
        ],
      },
      keepDiscriminatorProperty: true,
    },
  )
  request: RequestADto | RequestBDto;
}

class ResponseDataShapeA {
  @ApiProperty({ type: String, description: "1st field of data shape A" })
  shapeA1stField: string;

  @ApiProperty({ type: Boolean, description: "2nd field of data shape A" })
  shapeA2ndField: boolean;
}

class ResponseADto {
  @ApiProperty({ enum: [DiscriminatorType.A], description: "Discriminator type A" })
  type: DiscriminatorType.A;

  @ApiProperty({ type: ResponseDataShapeA, description: "Data of shape A" })
  data: ResponseDataShapeA;
}

class ResponseDataShapeB {
  @ApiProperty({ type: String, description: "1st field of data shape B" })
  shapeB1stField: string;

  @ApiProperty({ type: Boolean, description: "2nd field of data shape B" })
  shapeB2ndField: boolean;
}

class ResponseBDto {
  @ApiProperty({ enum: [DiscriminatorType.B], description: "Discriminator type B" })
  type: DiscriminatorType.B;

  @ApiProperty({ type: ResponseDataShapeB, description: "Data of shape B" })
  data: ResponseDataShapeB;
}

@ApiExtraModels(ResponseADto, ResponseBDto)
export class TestDiscriminatorsResponse {
  @ApiProperty({
    oneOf: [{ $ref: getSchemaPath(ResponseADto) }, { $ref: getSchemaPath(ResponseBDto) }],
    description: "Response data, which can be of shape A or shape B depending on the type",
    discriminator: {
      propertyName: "type",
      mapping: {
        [DiscriminatorType.A]: getSchemaPath(ResponseADto),
        [DiscriminatorType.B]: getSchemaPath(ResponseBDto),
      },
    },
  })
  data: ResponseADto | ResponseBDto;
}
```
