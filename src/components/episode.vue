<template>
    <div>
        <div class="text-center text-xl">
            <h1>Episoden Daten</h1>
        </div>

        <div class="mb-4 ml-5">
            <label class="block text-gray-700 text-sm font-bold mb-2" for="episodeid">Episode Id</label>
            <input
                v-model="episodeId"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="text"
                placeholder="Episode Id"
            />
        </div>

        <div class="mb-4 ml-5">
            <label class="block text-gray-700 text-sm font-bold mb-2" for="number">Number</label>
            <input
                v-model="number"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="integer"
                placeholder="Number of the episode"
            />
        </div>

        <div class="mb-4 ml-5">
            <label class="block text-gray-700 text-sm font-bold mb-2" for="title">Title:</label>
            <input
                v-model="title"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="text"
                placeholder="Title"
            />
        </div>
        <div class="mb-4 ml-5">
            <label class="block text-gray-700 text-sm font-bold mb-2" for="subtitle">Subtitle:</label>
            <input
                v-model="subtitle"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="text"
                placeholder="Subtitle"
            />
        </div>
        <div class="mb-4 ml-5">
            <label class="block text-gray-700 text-sm font-bold mb-2" for="summary">Summary:</label>
            <input
                v-model="summary"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="text"
                placeholder="Summary"
            />
        </div>
        <div class="mb-4 ml-5">
            <label class="block text-gray-700 text-sm font-bold mb-2" for="slug">Slug:</label>
            <input
                v-model="slug"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                type="text"
                placeholder="Slug"
            />
        </div>
        <br/>
        <button class="btn btn-blue" v-on:click="createEpisode">Create</button>
        <button class="btn btn-blue" v-on:click="getDataFromWordpress">Read data</button>
        <button class="btn btn-blue" v-on:click="setDataToWordpress">Write data</button>
        <button class="btn btn-blue" v-on:click="deleteEpisode">Delete</button>
        <button class="btn btn-blue" v-on:click="clearData">Clear</button>
        <br/> <br/>
        <input class="ml-5" type="file" @change="readFile" />
        <br/> <br/>
        <button class="btn btn-blue" v-on:click="getTranscriptToWordpress">Read Transcript</button>
        <button class="btn btn-blue" v-on:click="prevTranscriptToWordpress">Prev</button>
        <button class="btn btn-blue" v-on:click="nextTranscriptToWordpress">Next</button>
        <button class="btn btn-blue" v-on:click="setTranscriptToWordpress">Write Transcript</button>
        <button class="btn btn-blue" v-on:click="deleteTranscriptToWordpress">Delete Transcript</button>
        <br/>
        <br/>

        <table>
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Sprecher</th>
                    <th>Content</th>
                    <th>Content (Edit)</th>
                    <th>Start</th>
                    <th>End</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="transcript in transcripts">
                    <td>{{transcript.id}}</td>
                    <td>{{transcript.voice}}</td>
                    <td>{{transcript.text}}</td>
                    <td>
                        <input
                            v-model="transcript.text"
                            class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                            type="text"
                            placeholder="Content"
                        />
                    </td>
                    <td>{{transcript.start}}</td>
                    <td>{{transcript.end}}</td>
                </tr>
          </tbody>
        </table>


    </div>
</template>

<script>
export default {
    methods: {
        readFile(ev) {

            const file = ev.target.files[0];
            const reader = new FileReader();

            reader.onload = e => {
                this.file_content = e.target.result;
            }
            reader.readAsText(file);
        },
        createEpisode: function () {
            let url = 'http://localhost:10013/wp-json/podlove/v2/episodes';
            fetch(url, {
                method: 'POST',
                headers: {
                    'Authorization': auth
                }
            })
                .then(responce => responce.json())
                .then(data => { this.episodeId = data.id })
                .catch(err => console.log(err));
        },
        deleteEpisode: function () {
            let url = 'http://localhost:10013/wp-json/podlove/v2/episodes' + '/' + this.episodeId;
            fetch(url, {
                method: 'DELETE',
                headers: {
                    'Authorization': auth
                }
            })
                .then(responce => responce.json())
                .then(data => { this.episodeId = data.id })
                .catch(err => console.log(err));
        },
        setDataToWordpress: function () {
            let url = 'http://localhost:10013/wp-json/podlove/v2/episodes' + '/' + this.episodeId;

            let episodeData = {
                title: this.title,
                subtitle: this.subtitle,
            }
            if (this.number > 0)
                episodeData.number = this.number;
            if (this.summary.length > 0)
                episodeData.summary = this.summary;
            if (this.slug.length > 0)
                episodeData.slug = this.slug;

            fetch(url, {
                method: 'PATCH',
                headers: {
                    'Authorization': auth,
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(episodeData)
            })
                .then(responce => responce.json())
                .then(data => console.log(data))
                .catch(err => console.log(err));
        },
        getTranscriptToWordpress: function() {
            let url = 'http://localhost:10013/wp-json/podlove/v2/transcripts' + '/' + this.episodeId + '?';
            this.offset = 0;

            fetch(url + new URLSearchParams( {
                offset: this.offset,
                limit: this.limit,
            }))
                .then(responce => responce.json())
                .then(json => {
                    this.transcripts = json.transcript;
                })
                .catch(err => console.log(err));
        },
        prevTranscriptToWordpress: function() {
            let url = 'http://localhost:10013/wp-json/podlove/v2/transcripts' + '/' + this.episodeId + '?';

            this.offset = this.offset - this.limit;
            if (this.offset < 0)
                this.offset = 0;
            fetch(url + new URLSearchParams( {
                offset: this.offset,
                limit: this.limit,
            }))
                .then(responce => responce.json())
                .then(json => {
                    this.transcripts = json.transcript;
                })
                .catch(err => console.log(err));
        },
        nextTranscriptToWordpress: function() {
            let url = 'http://localhost:10013/wp-json/podlove/v2/transcripts' + '/' + this.episodeId + '?';

            this.offset = this.offset + this.limit;
            fetch(url + new URLSearchParams( {
                offset: this.offset,
                limit: this.limit,
            }))
                .then(responce => responce.json())
                .then(json => {
                    this.transcripts = json.transcript;
                })
                .catch(err => console.log(err));
        },
        setTranscriptToWordpress: function() {
            let url = 'http://localhost:10013/wp-json/podlove/v2/transcripts' + '/' + this.episodeId;

            if (this.file_content.length < 1) {
                alert("Please select a transcript before upload to wordpress");
                return;
            }

            let episodeData = {
                type: 'vtt',
                content: this.file_content,
            }
            fetch(url, {
                method: 'POST',
                headers: {
                    'Authorization': auth,
                    'Content-Type': 'application/json; charset=utf-8'
                },
                body: JSON.stringify(episodeData)
            })
                .then(responce => responce.json())
                .then(data => console.log(data))
                .catch(err => console.log(err));
        },
        deleteTranscriptToWordpress: function() {
            let url = 'http://localhost:10013/wp-json/podlove/v2/transcripts' + '/' + this.episodeId;

            fetch(url, {
                method: 'DELETE',
                headers: {
                    'Authorization': auth,
                    'Content-Type': 'application/json; charset=utf-8'
                },
            })
                .then(responce => responce.json())
                .then(data => console.log(data))
                .catch(err => console.log(err));
        },
        getDataFromWordpress: function () {
            let url = 'http://localhost:10013/wp-json/podlove/v2/episodes' + '/' + this.episodeId;
            fetch(url, { method: 'GET' })
                .then(response => response.json())
                .then(json => {
                    this.title = json.title;
                    this.subtitle = json.subtitle;
                    this.number = json.number;
                    this.summary = json.summary;
                })
                .catch(err => alert(err.message));
        },
        clearData: function () {
            this.title = '';
            this.subtitle = '';
            this.transcripts = '';
            this.summary = '';
            this.number = 0;
            this.file_content = '';
        }
    },
    data() {
        return {
            title: '',
            subtitle: '',
            summary: '',
            slug: '',
            number: 0,
            chapters: '',
            episodeId: 2,
            transcripts: '',
            file_content: '',
            offset: 0,
            limit: 5,
        }
    }
}
</script>

<style scoped>
.btn {
    @apply font-bold py-2 px-4 rounded ml-5;
}
.btn-blue {
    @apply bg-blue-500 text-white;
}
.btn-blue:hover {
    @apply bg-blue-700;
}

table {
  font-family: 'Open Sans', sans-serif;
  width: 1250px;
  border-collapse: collapse;
  border: 3px solid #44475C;
  margin: 10px 10px 0 10px;
}

table th {
  text-transform: uppercase;
  text-align: left;
  background: #44475C;
  color: #FFF;
  padding: 8px;
  min-width: 30px;
}

table td {
  text-align: left;
  padding: 8px;
  border-right: 2px solid #7D82A8;
}
table td:last-child {
  border-right: none;
}
table tbody tr:nth-child(2n) td {
  background: #D4D8F9;
}

</style>
