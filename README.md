# TalentBoard Backend

[![GitHub license](https://img.shields.io/github/license/Naereen/StrapDown.js.svg)](https://github.com/CodeOpsTechnologies)
[![Deployment](https://github.com/CodeOpsTechnologies/Talent-Board-Backend/actions/workflows/main.yml/badge.svg)](https://github.com/CodeOpsTechnologies/Talent-Board-Backend/actions/workflows/main.yml)
![coverage](coverage/badge-lines.svg)

This is a community initiative to connect active job seekers with organizations and people who participate in employee referral programs.

Currently, we encourage only job seekers / candidates who lost their jobs in the pandemic scenario. Referrals should be verified via LinkedIn profiles. (Click LinkedIn Connect-> Save profile as PDF-> Refer). As a community, it will be a win-win situation for all of us if we can support and make the future safe for one another. Help them to resume their career by referring. #refer2resume

## Architecture

![TalentBoard Architecture](https://user-images.githubusercontent.com/23396903/120200797-597e1f80-c242-11eb-9cc7-2c47126f088b.PNG)

The architecture diagram was built using [CloudSkew](https://cloudskew.com), a free architecture tooling application.

## Prerequisites
- [Node.js](https://nodejs.org/en/download/) & [NPM](https://www.npmjs.com/get-npm) 
- [Serverless framework](https://www.npmjs.com/package/serverless)
- [AWS Account](https://portal.aws.amazon.com/billing/signup#/start)

## Tech stack
- AWS Cloud
- NodeJS as the programming language

## AWS Services Consumed
- [AWS Lambda](https://aws.amazon.com/lambda/) for compute
- [Amazon API Gateway](https://aws.amazon.com/api-gateway/) to create REST APIs
- [Amazon Aurora Serverless](https://aws.amazon.com/rds/aurora/serverless/) with MySQL 5.7 engine as the database
- [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) for application monitoring and scheduled cron
- [Data API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html) for Aurora Serverless to connect to the RDS Cluster

## Serverless Framework Plugins Used
1.  [serverless-pseudo-parameters](https://www.npmjs.com/package/serverless-pseudo-parameters) to use CloudFormation Pseudo Parameters
2.  [serverless-dotenv-plugin](https://www.npmjs.com/package/serverless-dotenv-plugin) to preload environment variables into serverless
3.  [serverless-offline](https://www.npmjs.com/package/serverless-offline)

## Folder Structure
```bash
.
├── __tests__        # Automated tests and test utils using JEST framework
├── ApiDocs          # Autogenerated Documentation files
├── Docs             # API Documentation source files
├── Locations        # Locations module source files to get countries, states and cities
├── Resources        # Database and Lambda configuration files (essentially serverless templates which will deploy all resources)
├── SampleRequests   # Sample request JSON files to test lambda functions locally
├── TalentBoard      # Source files for all talent board APIs 
├── TalentBoardCron  # Source files for cron job to expire user profiles based on their retention period provided
├── Utils            # Utilities such as constants, reusable functions, database connection
├── LICENSE
└── README.md
```

## Getting Started
[![GitHub code size in bytes](https://img.shields.io/github/languages/code-size/CodeOpsTechnologies/talent-board-be?logo=github)](https://talent.awsug.in/) 
[![GitHub commit activity](https://img.shields.io/github/commit-activity/m/CodeOpsTechnologies/talent-board-be?color=bluevoilet&logo=github)](https://github.com/CodeOpsTechnologies/talent-board-be/commits/) 
[![GitHub repo size](https://img.shields.io/github/repo-size/CodeOpsTechnologies/talent-board-be?logo=github)](https://talent.awsug.in/)

**1.** Fork this repository and clone into your local machine.
```bash
git clone https://github.com/CodeOpsTechnologies/talent-board-be.git
```
**2.** Navigate to the local project directory.
```bash
cd talent-board-be
```
**3.** Install all associated npm dependencies.
```bash
npm install
```
**4.** You'll need to navigate into *[/Resources/DatabaseResources](https://github.com/CodeOpsTechnologies/talent-board-be/tree/main/Resources/DatabaseResources)* and run the following command to deploy the Aurora Serverless database **[Note: This step is only needed to do once manually]**:
```bash
cd Resources/DatabaseResources/
# Provision database resources (VPC, Subnets, RDS Cluster, etc.)
serverless deploy
```
The above command will deploy the following resources:
1.  Aurora Serverless RDS Cluster
2.  Aurora Serverless cluster parameter group
3.  Secrets manager to store the DB credentials
4.  Public route table
5.  Three subnets
6.  VPC network
7.  Security group

The above command will output the following details:
1.  `SecretARN` - The Secrets Manager's ARN
2.  `ClusterName` - The Aurora RDS cluster's name
3.  `DBName` - The name of the database created

**5.** Create a secret in your GitHub (https://docs.github.com/en/actions/reference/encrypted-secrets) repo to store the name of the just created database stack name. You can create one by navigating to the Settings and selecting the `Secrets` in the options.

`DATABASE_STACK_NAME=talent-board-backend-database`

The above environment variable will be referenced in the serverless API project to add the database's stack ouputs as environment variables to the lambda functions 

**6.** The lambda function's code is bundled before packaging and deploying to the AWS account using [webpack](https://webpack.js.org/):
```shell
npm run bundle
```

**7.** Navigate to the root of your project directory and run the following command to deploy the REST APIs and corresponding Lambda functions:
```bash
# Provision the required API Gateway endpoints & Lambda functions
cd ..
serverless deploy
```

After performing all the above steps, the DB, API and Lambda function resources would be deployed to your AWS account

**8.** Connect to RDS cluster:

To proceed further, you will need to connect to the newly created RDS cluster using the [query editor](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/query-editor.html) on the AWS console.
- Select the RDS cluster created in step 1 from the dropdown. It would be of the format `talent-board-backend-database-<stage>-aurorardscluster-<cluster name>`
- You can connect to the database using the Secrets Manager. You can get the ARN from the output of step 1 (`SecretARN`). By doing so, AWS will automatically fetch the DB username and password from the Secrets Manager and connect with the database
- Enter `TB` for the name of the database

![Connect to Database](https://i.ibb.co/cybcXLq/connect-to-database.png)

**9.** Navigate to *[Resources/DatabaseResources/createTable.sql](https://github.com/CodeOpsTechnologies/talent-board-be/blob/main/Resources/DatabaseResources/createTable.sql)* and paste the create table script into query editor

The table schema:
- `name` - VARCHAR(256) NOT NULL -  Name of the user adding the profile
- `skills` - JSON NOT NULL - A list of skills selected by the user
- `industry` - VARCHAR(256) NOT NULL - The industry the user belongs to
- `jobRole` - VARCHAR(256) NOT NULL - The user's current job role
- `proficiencyLevel` - VARCHAR(18) NOT NULL - The user's AWS proficiency level
- `visibilityDuration` - TINYINT(4) NOT NULL - The visibility duration until which the user wants their profile to be visible on the Talent Board page. The accepted values are `1 -> 15 days, 2 -> 30 days, 3 -> 45 days, 4 -> 60 days, 5 -> 90 days`
- `relocation` - TINYINT(1) NOT NULL DEFAULT `0` - A boolean flag to indicate if the user is willing to relocate
- `state` - VARCHAR(128) NOT NULL - User's state
- `city` - VARCHAR(128) NOT NULL - User's city
- `experience` - TINYINT(10) NOT NULL - User's experience in number of years. The accepted values are - `1 -> 0-1 year, 2 -> 1-3 years, 3 -> 4 - 7 years, 4 -> 8-10 years, 5 -> 11-15 years, 6 -> 16+ years`
- `linkedinUrl` - TEXT CHARACTER SET utf8 NOT NULL - User's LinkedIn URL
- `createdAt` - TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP - The UTC timestamp at which the profile was created
- `updatedAt` - TIMESTAMP NULL DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP - The UTC timestamp at which the user's profile was updated
- `expireAfter` - DATE NOT NULL - The UTC date after which the profile should be marked as expired. This date is calculated based on the `visibilityDuration` provided by the user
- `profileStatus` - TINYINT(4) DEFAULT `1` - The status of the profile, 1 -> Active, 2 -> Expired 

**10.** To deploy the API docs to your AWS account, follow the steps given below:
```shell
npm run apidoc # This will build the API documentation - creates the static HTML, CSS etc. 
cd ApiDocs
serverless --verbose # Deploys api documentation to your aws account
cd ..
```

The above steps will deploy the statically generated API docs by running the `npm run apidoc` and the same gets deployed to an S3 bucket using the [Serverless Components](https://github.com/serverless-components/website)

## Points to Remember:
1.  The application is built to have 2 stages/environments - dev (on the `main` branch) and prod (on the `prod` branch)
2.  Aurora Serverless supports DB Auto Pause (https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.how-it-works.html), where you configure to pause your Aurora Serverless v1 DB cluster after a given amount of time with no activity. The setup in this application has the dev environment [configured](https://github.com/CodeOpsTechnologies/talent-board-be/blob/main/Resources/DatabaseResources/AuroraRDSCluster.yml#L35) with DB auto pause enabled if there is no activity for more than 10 minutes. But the same is disabled for the prod environment because it takes close to 30 seconds for the DB to be up and running once it gets into the paused state
3.  The application is configured to be deployed into two AWS regions based on the stage:
    - dev environment: Singapore (`ap-southeast-1`)
    - prod environment: Mumbai (`ap-south-1`)

## GitHub Actions
### Steps to setup CI/CD using GitHub Actions
The following is the list of steps that is executed as a part of the CICD setup:

1. Store all account credentials in GitHub [secrets](https://docs.github.com/en/actions/reference/encrypted-secrets) to access them in environment without exposing. The AWS credentials will be used for deploying the application into your AWS account and the database details is used for running the test cases as a part of the CICD setup
```bash
AWS_ACCESS_KEY_ID: ${{secrets.AWS_ACCESS_KEY_ID}}
AWS_SECRET_ACCESS_KEY: ${{secretAWS_SECRET_ACCESS_KEY}}
DATABASE_STACK_NAME: ${{secrets.DATABASE_STACK_NAME}}
SECRET_ARN: ${{secrets.SECRET_ARN}}
CLUSTER_ARN: ${{secrets.CLUSTER_ARN}}
REGION: ${{secrets.REGION}}
DB_NAME: ${{secrets.DB_NAME}}

```
2. Install serverless
3. Install npm
4. Tests - Automated tests to execute while deploying
5. Deploy API DOCS
6. Build project using webpack
7. Deploy to dev environment: Singapore (`ap-southeast-1`)
8. Deploy to prod environment: Mumbai (`ap-south-1`)

## Testing
### Steps to perform testing of deployed APIs locally using serverless framework
1. Navigate to directory *[/SampleRequests](https://github.com/CodeOpsTechnologies/Talent-Board-Backend/tree/main/SampleRequests)*
2. There are 3 sample request files in `json` format.
    - [createProfile.json](https://github.com/CodeOpsTechnologies/Talent-Board-Backend/blob/main/SampleRequests/createProfile.json) - to test addProfile API
    - [getFilters.json](https://github.com/CodeOpsTechnologies/Talent-Board-Backend/blob/main/SampleRequests/getFilters.json) - to test getFilters API
    - [listProfiles.json](https://github.com/CodeOpsTechnologies/Talent-Board-Backend/blob/main/SampleRequests/listProfiles.json) - to test listProfiles API
3. Go to the terminal and run the following command:
```shell
serverless invoke local -f talentBoard -p ./SampleRequests/<requestFile.json> --stage <stage>
# ex - serverless invoke local -f talentBoard -p ./SampleRequests/listProfiles.json --stage dev
```
The command above will print response from the respective api in terminal window.

## Contribution Guidelines
[![GitHub pull requests](https://img.shields.io/github/issues-pr-raw/CodeOpsTechnologies/talent-board-be?logo=git&logoColor=white)](https://github.com/CodeOpsTechnologies/talent-board-be/compare) 
[![GitHub contributors](https://img.shields.io/github/contributors/CodeOpsTechnologies/talent-board-be?logo=github)](https://github.com/CodeOpsTechnologies/talent-board-be/graphs/contributors) 
[![Author](https://img.shields.io/badge/Author-@CodeOpsTechnologies-gray.svg?colorA=gray&colorB=dodgerblue&logo=github)](https://github.com/CodeOpsTechnologies/)

- Write clear meaningful git commit messages (Do read [this](http://chris.beams.io/posts/git-commit/)).
- Make sure your PR's description contains GitHub's special keyword references that automatically close the related issue when the PR is merged. (Check [this](https://github.com/blog/1506-closing-issues-via-pull-requests) for more info)
- When you make very minor changes to a PR of yours (like for example fixing a typo in documentation, minor changes requested by reviewers) make sure you squash your commits afterward so that you don't have an absurd number of commits for a very small fix. (Learn how to squash at [here](https://davidwalsh.name/squash-commits-git))
- Always create PR to `main` branch.
- Please read our [Code of Conduct](./CODE_OF_CONDUCT.md).
- Refer [this](https://github.com/CodeOpsTechnologies/talent-board-fe/blob/master/CONTRIBUTING.md) for more.

## Other Useful Links

Frontend Code - [https://github.com/CodeOpsTechnologies/talent-board-fe](https://github.com/CodeOpsTechnologies/talent-board-fe)
<br />
Backend Code - [https://github.com/CodeOpsTechnologies/Talent-Board-Backend](https://github.com/CodeOpsTechnologies/Talent-Board-Backend)
<br />
API Documentation - [http://talentboard-api-documentation.s3-website-ap-southeast-1.amazonaws.com/](http://talentboard-api-documentation.s3-website-ap-southeast-1.amazonaws.com/)

***Glad to see you here! Show some love by [starring](https://github.com/CodeOpsTechnologies/talent-board-fe/) this repo.***

[![Facebook](https://img.shields.io/static/v1.svg?label=connect&message=@CodeOpsTech&color=grey&logo=facebook&style=flat&logoColor=white&colorA=royalblue)](https://www.facebook.com/CodeOpsTech)
[![LinkedIn](https://img.shields.io/static/v1.svg?label=connect&message=@CodeOpsTech&color=grey&logo=linkedin&style=flat&logoColor=white&colorA=royalblue)](https://www.linkedin.com/company/codeops-technologies/)
[![Twitter](https://img.shields.io/static/v1.svg?label=connect&message=@CodeOpsTech&color=grey&logo=twitter&style=flat&logoColor=white&colorA=royalblue)](https://twitter.com/CodeOpsTech)
