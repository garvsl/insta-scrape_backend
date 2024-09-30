# insta-scrape_backend

The insta-scrape_backend for insta-scrape_email, in order to create anything for the dbs, u can use localhost:8000/docs, u can also run the scraper via a celery background task. redis-server is used to cache the logins and retrievals, and hold the celery task number.

u will need session id as a header in order to access any route, make sure to save that or u will need prisma studio to find it

there is no method to create a user u need to seed the db with one first.

# run

run in one terminal

cd app

python3 main.py

run in seperate terminal

cd app

redis-server

run in seperate terminal

cd app

celery -A app.task worker -l INFO -f celery.logs

# setup

.env:
DATABASE_URL=

python3 -m venv venv

source venv/bin/activate

pip3 install -r requirements.txt

prisma generate

# sample typescript seed

import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcrypt';
import fs from 'fs';

const prisma = new PrismaClient();

async function main() {
const users = [
{
email: '',
name: '',
password: '',
},
];

users.map(async (user) => {
const hashedPassword = await bcrypt.hash(user.password, 10);

    await prisma.user.upsert({
      where: { email: user.email },
      update: {},
      create: {
        name: user.name,
        email: user.email,
        hashedPassword,
        Script: {
          create: {
            title: '',
            body: `Hi {},

                lorem ipsum

                Best regards

${user.name}`,
},
},
},
});
});

const influencersJson = JSON.parse(
fs.readFileSync('influencers.json', 'utf-8'),
);

#create influencer here, json format
const influencers = influencersJson.map(
(inf: { handle: string; email: string; name: string; created: string }) => {
return {
handle: inf.handle,
email: inf.email,
name: inf.name,
created: new Date(inf.created),
};
},
);

await prisma.influencer.createMany({
data: influencers,
skipDuplicates: true,
});

    #not really needed

const usernamesJson = JSON.parse(fs.readFileSync('usernames.json', 'utf-8'));
const usernames = usernamesJson.map(
(inf: {
id: string;
handle: string;
created: string;
checked: boolean;
}) => {
return {
id: inf.id,
handle: String(inf.handle),
checked: inf.checked,
created: new Date(inf.created),
};
},
);

await prisma.username.createMany({
data: usernames,
skipDuplicates: true,
});
}

main()
// eslint-disable-next-line promise/always-return
.then(async () => {
await prisma.$disconnect();
  })
  .catch(async (e) => {
    console.error(e);
    await prisma.$disconnect();
process.exit(1);
});
