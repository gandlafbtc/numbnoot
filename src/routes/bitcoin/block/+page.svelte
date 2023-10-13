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
    <div>
        <div class="bg-primary text-center rounded-t-lg">
            Block
        </div>
        <div class="flex border border-primary p-3 rounded-b-lg">
            <div class="form-control">
                <label class="input-group">
                    <span>Nonce</span>
                    <input type="text" placeholder="000000" class="input input-bordered" />
                </label>
                <label class="input-group">
                    <span>Block header</span>
                    <input type="text" placeholder="000000" class="input input-bordered" />
                </label>
            </div>
        </div>
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