<script lang="ts">
	import { BlindedMessage, SigningAuthority } from '@gandlaf21/blind-signature';
	import type { Point } from '@noble/secp256k1';
	import { bytesToHex, randomBytes } from '@noble/hashes/utils';
	
	const signingAuth = new SigningAuthority();
	
	type Wallet = {
		id: number;
		bms: { bm: BlindedMessage; B_: Point }[];
		ecash: { secret: Uint8Array; signature: Point }[];
		redeemed: number
	};
	
	let mintBurns:string[] = [];
	let wallets: Wallet[] = [
		{ id: 0, bms: [], ecash: [], redeemed: 0 },
		{ id: 1, bms: [], ecash: [],redeemed: 0 },
		{ id: 2, bms: [], ecash: [],redeemed: 0 }
	];

	let mintBMs: { bm: Point; walletID: number }[] = [];

	const mint = async (id: number) => {
		const wallet = wallets.find((w) => w.id === id);

		const bm = new BlindedMessage();
		const B_ = await bm.createBlindedMessage(randomBytes(16));
		wallet?.bms.push({ bm, B_ });
		mintBMs.push({ bm: B_, walletID: id });
		wallets = [...wallets];
		mintBMs = [...mintBMs];
	};

	const sign = () => {
		if (mintBMs.length) {
			const C_ = signingAuth.createBlindSignature(mintBMs[0].bm);
			const unblinded = wallets[mintBMs[0].walletID].bms
				.find((bm) => bm.B_.toHex(true) === mintBMs[0].bm.toHex(true))
				?.bm.unblindSignature(C_, signingAuth.publicKey);
			if (!unblinded?.secret) {
				return;
			}
			wallets[mintBMs[0].walletID].ecash.push({ secret: unblinded.secret, signature: unblinded.C });
			wallets = [...wallets];
		}
		mintBMs.shift();
		mintBMs = [...mintBMs];
	};
	const decline = () => {
		mintBMs.shift();
		mintBMs = [...mintBMs];
	};

	const formattedString = (string: string): string => {
		return string.substring(0, 4) + '...' + string.substring(string.length - 5, string.length - 1);
	};

	const send = (ecash:{ secret: Uint8Array; signature: Point } , walletID: number) => {
		if (walletID=== -1) {
			walletID=2
		}
		else if (walletID=== 3) {
			walletID=0
		}
		
		wallets[walletID].ecash.push(ecash)
		wallets = [...wallets];
	}
	const redeem = async (ecash:{ secret: Uint8Array; signature: Point }, walletID: number) => {
		const aY = await signingAuth.calculateCVerify(ecash.secret)
		
		if (!signingAuth.verify(aY,ecash.signature)) {
			return 
		}
		if (mintBurns.includes(bytesToHex(ecash.secret))) {
			return
		}
		mintBurns.push(bytesToHex(ecash.secret))
		mintBurns= [...mintBurns];

		wallets[walletID].redeemed++
		wallets = [...wallets];

	}
	const reset = () => {
		wallets =[
		{ id: 0, bms: [], ecash: [], redeemed: 0 },
		{ id: 1, bms: [], ecash: [],redeemed: 0 },
		{ id: 2, bms: [], ecash: [],redeemed: 0 }
	]

	mintBMs = []
	mintBurns = []
	}
</script>

<div class="flex flex-col gap-5">
<div class="flex justify-center flex-col gap-5 items-center">
	<span class=" text-transparent font-bold text-2xl bg-clip-text bg-gradient-to-r from-primary to-secondary">Wallets mint interactions</span>
	<div>
		<button class="btn btn-secondary" on:click={reset}>reset</button>
	</div>
</div>

<div class="grid grid-cols-3 gap-2">
	{#each wallets as w}
		<div>
			<div class="bg-primary rounded-t-lg text-center p-1">
				----------------------------------------------- Wallet {w.id}
				-----------------------------------------------
			</div>
			<div class="border border-primary p-3 rounded-b-lg h-96 flex flex-col justify-between">
				<div class="flex gap-2 w-full">
					<div class="w-56">
						<div class="bg-primary rounded-t-lg text-center p-1">my BMs</div>
						<div
							class="border border-primary p-3 rounded-b-lg h-32 flex flex-col justify-between gap-1 overflow-y-auto"
						>
							{#each w.bms as bm, i}
								<div class="flex">
									<div class="bg-primary rounded-l-lg p-1">B_</div>
									<p class="border border-primary rounded-r-lg p-1">
										{formattedString(bm.B_.toHex(true))}
									</p>
								</div>
							{/each}
						</div>
					</div>
					<div class="w-[30em]">
						<div class="bg-primary rounded-t-lg text-center p-1">my ecash</div>
						<div
							class="border border-primary p-3 rounded-b-lg h-32 flex flex-col justify-between overflow-y-auto gap-1"
						>
							{#each w.ecash as ecash, i}
								<div class="flex gap-1">
									<div class="flex">
										<div class="bg-primary rounded-l-lg p-1">C</div>
										<p class="border border-primary rounded-r-lg p-1">
											{formattedString(ecash.signature.toHex(true))}
										</p>
									</div>

									<div class="flex">
										<div class="bg-primary rounded-l-lg p-1">x</div>
										<p class="border border-primary rounded-r-lg p-1">
											{formattedString(bytesToHex(ecash.secret))}
										</p>
									</div>
									<div>
										<button class="btn btn-xs btn-primary" on:click={()=>send(ecash, w.id-1)}>👈send
										</button>
										<button class="btn btn-xs btn-primary" on:click={()=>send(ecash, w.id+1)}>send👉</button>
										<button class="btn btn-xs btn-secondary" on:click={()=>redeem(ecash, w.id)}>redeem</button>
									</div>
								</div>
							{/each}
						</div>
					</div>
				</div>
				<div>
					<button class="btn btn-primary" on:click={() => mint(w.id)}>mint</button>
					<p>redeemed: {w.redeemed} tokens</p>
				</div>
			</div>
		</div>
	{/each}

	<div class="col-span-3 w-full">
		<div class="bg-secondary rounded-t-lg text-center p-1">
			----------------------------------------------- mints
			-----------------------------------------------
		</div>
		<div class="border border-secondary p-3 rounded-b-lg h-96 flex flex-col justify-between">
			<div class="flex gap-2 w-full">
				<div class="w-full">
					<div class="bg-secondary rounded-t-lg text-center p-1">incoming messages</div>
					<div
						class="border border-secondary p-3 rounded-b-lg h-32 flex flex-col justify-between overflow-y-auto gap-1" 
					>
						{#each mintBMs as bm, i}
							<div class="flex">
								<div class="bg-secondary rounded-l-lg p-1">B_</div>
								<p class="{i === 0 ? 'font-bold' : ''} border border-secondary rounded-r-lg p-1">
									{formattedString(bm.bm.toHex(true))}
								</p>
							</div>
						{/each}
					</div>
				</div>
				<div class="w-full">
					<div class="bg-secondary rounded-t-lg text-center p-1">burned secrets</div>
					<div
						class="border border-secondary p-3 rounded-b-lg h-32 flex flex-col justify-between gap-1 overflow-y-auto">
						{#each mintBurns as s, i}
						<div class="flex">
							<div class="bg-secondary rounded-l-lg p-1">x</div>
							<p class="border border-secondary rounded-r-lg p-1">
								{formattedString(s)}
							</p>
						</div>
					{/each}
					
					</div>
				</div>
			</div>
			<div>
				<button
					class="btn {mintBMs.length ? 'btn-secondary' : 'btn-disabled'} "
					on:click={() => sign()}>sign</button
				>
				<button
					class="btn {mintBMs.length ? 'btn-warning' : 'btn-disabled'} "
					on:click={() => decline()}>decline</button
				>
			</div>
		</div>
	</div>
</div>
</div>
