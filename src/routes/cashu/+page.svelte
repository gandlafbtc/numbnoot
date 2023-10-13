<script lang="ts">
	import Step1 from '../../comps/cashu/Step1.svelte';
	import { BlindedMessage, SigningAuthority } from '@gandlaf21/blind-signature';
	import { hexToBytes } from '@noble/hashes/utils';

	import type { Point } from '@noble/secp256k1';

	const signingAuth = new SigningAuthority();
	let step = 1;
	let secret = '';
	let bm: BlindedMessage;
	let B_: Point;
	let C_: Point;
	let C: Point;
	let isValid: boolean;
	let aY: Point;

	let seenSecrets: string[] = [];
	const chooseSecret = () => {
		step = 2;
	};

	const createBM = async () => {
		bm = new BlindedMessage();
		B_ = await bm.createBlindedMessage(hexToBytes(secret));
		step = 3;
	};

	const sign = () => {
		C_ = signingAuth.createBlindSignature(B_);
		step = 5;
	};
	const unblind = () => {
		C = bm.unblindSignature(C_, signingAuth.publicKey).C;
		step = 7;
	};

	const verify = async () => {
		aY = await signingAuth.calculateCVerify(hexToBytes(secret));
		console.log(C.toHex(true));
		console.log(aY.toHex(true));
		if (aY.toHex(true) === C.toHex(true) && !seenSecrets.includes(secret)) {
			seenSecrets.push(secret);
			seenSecrets = [...seenSecrets];
			isValid = true;
		} else {
			isValid = false;
		}
		step = 9;
	};
	const reset = () => {
		step = 1;
		secret = '';
		bm = new BlindedMessage();
		isValid = false;
	};
</script>


<div class="flex flex-col items-center justify-center gap-10">
	<span class=" text-transparent font-bold text-2xl bg-clip-text bg-gradient-to-r from-primary to-secondary">Cashu Protocol step by step</span>
	<div>
		<button class="btn btn-secondary" on:click={reset}> reset </button>
	</div>

	<div class="h-60">
		<div class="{[1,2,3,6,7].includes(step)? 'bg-primary':'bg-secondary'} rounded-t-lg text-center p-1">
			explanation
		</div>
		<div class="border {[1,2,3,6,7].includes(step)? 'border-primary':'border-secondary'} p-5 rounded-b-lg w-92 flex flex-col justify-between">
			{#if step ===1}
			<p>
				The first step to create a cashu token is to choose a random <span class="text-error">secret (x)</span>.
			</p>
			<p>
				The secret has to be impossible to guess or brute-force.
			</p>
<p>
	Click on 'generate' to create a random <span class="text-error">secret (x)</span> and 'choose this secret' once you're ready to proceed.  
</p>
			{/if}
			<div class="flex flex-col gap-2">
			{#if step ===2}

				<p>
					The next step is to derive a <span class="text-info">blind message (B_)</span> from the secret we created. This involves a few steps: 
				</p>
				<ol class="list-decimal">
					<li>
						derive point <span class="text-error">Y</span>, representing <span class="text-error">secret (x)</span> on the secp256k1 elliptic curve  
					</li>
					<li>
						generate a random private key as a <span class="text-success">blinding factor (r)</span>   
					</li>
					
					<li>
						Compute the <span class="text-info">blinded message (B_)</span> by performing <span class="text-error">Y</span> + <span class="text-success">r</span>G   = <span class="text-info">B_</span>   
					</li>
				</ol> 
				<p>
					clicking on 'create <span class="text-info">blinded message</span> from secret' will follow these steps to create a <span class="text-info">blinded message (B_)</span> 
				</p>
			{/if}
			{#if step ===3}
			<p>
				Now that we have created a <span class="text-info">blinded message</span>, we can send id to the mint to receive a signature on <span class="text-info">B_</span>
			</p>
			{/if}
			{#if step ===4}
			<p>
				If the mint receives a request for a  <span class="text-warning">blind signature (C_)</span>, they usually want to make sure that the provided blinded message is eligible for a signature in the first place.
			</p>
			<p>
				This means in the context of cashu, that the user has to provide either a valid cashu token, or proof of payment of a lightning invoice they requested beforehand from the mint.
			</p>
			<p>
				If everything checks out, the mint can sign with the mints private key <span class="text-accent">private key (k)</span>: 
			</p>
			<p>
				<span class="text-accent">k</span><span class="text-info">B_</span>= <span class="text-warning">C_</span>
			</p>
			{/if}
			{#if step ===5}
			<p>
				after signing, the mint can send the <span class="text-warning">blind signature (C_)</span> back to the wallet
			</p>

			{/if}
			{#if step ===6}
			<p>
				When the wallet receives the <span class="text-warning">blind signature (C_)</span>, it has to 'unblind' it. For this process, we require the <span class="text-success">blinding factor (r)</span> that we initially used for blinding, and the mints <span class="text-secondary">public key (K)</span>
			</p>
			<p>
				<span class="text-warning">C_</span>-<span class="text-success">r</span><span class="text-secondary">K</span>= <span class="text-primary">C</span>
			</p>
			<p class="font-bold">
				<span class="text-error">secret (x)</span> together with <span class="text-primary">signature (C)</span> is the ecash!
			</p>
			{/if}
			{#if step ===7}
			<p>
				If the wallet wants to redeem the ecash (for let's say paying a lightning invoice through the mint), it will send the <span class="text-error">secret (x)</span>  and and the <span class="text-primary">signature (C)</span> to the mint 
			</p>

			{/if}
			{#if step ===8}
			<p>
				the mint can now calculate kY by first deriving Y from the <span class="text-error">secret (x)</span> like we did in the first step, and multiplying it with its <span class="text-accent">private key (k)</span>. 
			</p>
			<p>
				if <span class="text-accent">k</span><span class="text-error">Y</span>==<span class="text-primary">C</span> , then the mint knows it must've signed the <span class="text-error">secret (x)</span>, but the mint cannot determine which <span class="text-info">blinded message (B_)</span> the <span class="text-primary">signature (C)</span> corresponds to (assuming there is more than one blinded message in the system)
			</p>

			<p>
			Lastly, the mint checks against its database of spent <span class="text-error">secrets (x)</span> if the secret has already been spent. If it's not in the database, the mint adds it. If it's already in the database, the ecash is considered invalid.
			</p>
			{/if}
			{#if step ===9}
			<p>
				that's it! click on reset to try again. 
			</p>
			{/if}
		</div>
	</div>
	</div>

	<div class="flex gap-3">
		<div>
			<div class="bg-primary rounded-t-lg text-center p-1">
				-------------------------------------------------------------------------------- Wallet
				--------------------------------------------------------------------------------
			</div>
			<div class="border border-primary p-3 rounded-b-lg h-96 flex flex-col justify-between">
				<div>
					{#if step === 1}
						<div class="flex flex-col gap-2">
							<Step1 bind:secret />
							<button on:click={chooseSecret} class="btn {secret ? 'btn-primary' : 'btn-disabled'}">
								choose this <span class="text-error">secret</span>
							</button>
						</div>
					{:else if step === 2}
						<button on:click={createBM} class="btn btn-primary">
							Create blinded Message from <span class="text-error">secret</span>
						</button>
					{:else if step === 3}
						<button
							on:click={() => {
								step = 4;
							}}
							class="btn btn-primary"
						>
							send blinded message to mint
						</button>
					{:else if step === 6}
						<button class="btn btn-primary" on:click={unblind}> unblind signature </button>
					{:else if step === 7}
						<button
							class="btn btn-primary"
							on:click={() => {
								step = 8;
							}}
						>
							redeem at mint
						</button>
					{/if}
				</div>
				<div class="flex flex-col gap-1">
					<div>Local data:</div>
					{#if step > 1}
						<p>
							<span class="text-error">secret (x) : {secret}</span>
						</p>
					{/if}
					{#if step > 2}
						<p>
							<span class="text-error">secret on curve (Y): {bm.Y.toHex(true)}</span>
						</p>
						<p><span class="text-success">blinding factor (r): {bm.r}</span></p>
						<p><span class="text-info">Blinded Message (B_): {B_.toHex(true)}</span></p>
					{/if}
					{#if step > 5}
						<p>
							<span class="text-warning">blind Signature (C_): {C_.toHex(true)}</span>
							
						</p>
					{/if}
					{#if step > 6}
						<p>
							<span class="text-primary">unblinded Signature (C): {C.toHex(true)}</span>
						</p>
					{/if}
				</div>
			</div>
		</div>

		<div class="flex flex-col gap-2">
			<div>
				<div class="bg-secondary rounded-t-lg text-center p-1">
					-------------------------------------------------------------------------------- Mint
					--------------------------------------------------------------------------------
				</div>
				<div class="border border-secondary p-3 rounded-b-lg h-96 flex flex-col justify-between">
					<div>
						{#if step === 4}
							<button class="btn btn-secondary" on:click={sign}> Sign Blinded Message </button>
						{:else if step === 5}
							<button
								class="btn btn-secondary"
								on:click={() => {
									step = 6;
								}}
							>
								Send signature to Wallet
							</button>
						{:else if step === 8}
							<button class="btn btn-secondary" on:click={verify}> verify </button>
						{/if}
					</div>
					<div class="flex flex-col gap-1">
						<div>Local data:</div>

						<p><span class="text-secondary">Mint publicKey (K): {signingAuth.publicKey.toHex(true)}</span></p>

						{#if step > 3}
							<p><span class="text-info">Blinded Message (B_): {B_.toHex(true)}</span></p>
						{/if}
						{#if step > 4}
						
							<p> <span class="text-warning">Blind Signature (C_): {C_.toHex(true)}</span></p>
						{/if}
						{#if step > 7}
							<p><span class="text-error">secret (x): {secret}</span></p>
							<p><span class="text-primary">signature (C): {C.toHex(true)}</span></p>
						{/if}
						{#if step > 8}
							<p>verification (<span class="text-accent">k</span><span class="text-error">Y</span>): <span class=" text-transparent bg-clip-text bg-gradient-to-r from-accent to-error">{aY.toHex(true)}</span></p>
							<p>is valid? (kY==C) AND is not in DB: {isValid}</p>
						{/if}
					</div>
				</div>
			</div>
			<div>
				<div class="bg-secondary rounded-t-lg text-center p-1">
					---------------- seen secrets ----------------
				</div>
				<div class="border border-secondary p-3 rounded-b-lg h-40 flex flex-col justify-start">
					{#each seenSecrets as secret}
						<p><span class="text-error">{secret}</span></p>
					{/each}
				</div>
			</div>
		</div>
	</div>
</div>
