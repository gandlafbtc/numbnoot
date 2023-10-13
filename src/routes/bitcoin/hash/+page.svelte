<script>
	import { bytesToHex } from "@noble/hashes/utils";
	import Cogs from "../../../comps/Cogs.svelte";
	import {sha256} from "@noble/hashes/sha256";
    let input = ''
    let hash =''
    const doHash = () => {
        hash = bytesToHex(sha256(input)) 
    }
</script>
<div class="flex flex-col items-center gap-2 ">
    <div class="join">
        <input bind:value={input} type="text" class="input-primary input join-item" placeholder="any data....">
        <button class="btn btn-primary join-item" on:click={doHash}>hash!</button>
    </div>
    <div>
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
            <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 13.5L12 21m0 0l-7.5-7.5M12 21V3" />
          </svg>
    </div>
    <div>
        <div class="bg-primary text-center rounded-t-lg">
            sha256
        </div>
        <div class="border-primary rounded-b-lg border p-3">
            <Cogs></Cogs>
        </div>
    </div>
    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
        <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 13.5L12 21m0 0l-7.5-7.5M12 21V3" />
      </svg>

    <textarea class="textarea input-primary" placeholder="hash will appear here" readonly bind:value={hash} />
</div>