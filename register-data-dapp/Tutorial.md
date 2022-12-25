## Perquisites

- yarn
- vscode

## Follow along

- `yarn create react-app register-data-dapp --template typescript`
- `cd register-data-dapp`
- `code .`
- `yarn start` - will open the browser

### Add Dependencies

- `yarn add @mui/material @emotion/react @mui/icons-material @emotion/styled @concordium/web-sdk @concordium/browser-wallet-api-helpers`

### Connect

- Add a component [`Header.tsx`](./src/Header.tsx) with following code

```tsx
import {
	detectConcordiumProvider,
	WalletApi,
} from "@concordium/browser-wallet-api-helpers";
import { AppBar, Toolbar, Typography, Button } from "@mui/material";
import { useState } from "react";

export default function Header(props: {
	onConnected: (provider: WalletApi, account: string) => void;
	onDisconnected: () => void;
}) {
	const [isConnected, setConnected] = useState(false);

	function connect() {
		detectConcordiumProvider()
			.then((provider) => {
				provider
					.connect()
					.then((account) => {
						setConnected(true);
						props.onConnected(provider, account!);
					})
					.catch((_) => {
						alert("Please allow wallet connection");
						setConnected(false);
					});
				provider.removeAllListeners();
				provider.on("accountDisconnected", () => {
					setConnected(false);
					props.onDisconnected();
				});
				provider.on("accountChanged", (account) => {
					props.onDisconnected();
					props.onConnected(provider, account);
					setConnected(true);
				});
				provider.on("chainChanged", () => {
					props.onDisconnected();
					setConnected(false);
				});
			})
			.catch((_) => {
				console.error(`could not find provider`);
				alert("Please download Concordium Wallet");
			});
	}

	return (
		<AppBar>
			<Toolbar>
				<Typography variant="h6" component="div" sx={{ flexGrow: 1 }}>
					Concordium Register Data Dapp
				</Typography>
				<Button color="inherit" onClick={connect} disabled={isConnected}>
					{isConnected ? "Connected" : "Connect"}
				</Button>
			</Toolbar>
		</AppBar>
	);
}
```

- Update the [`App.tsx`](./src/App.tsx) to use the newly created component

```tsx
import "./App.css";
import Header from "./Header";

export default function App() {
	return (
		<div className="App">
			<Header
				onConnected={(_provider, account) => alert(`connected ${account}`)}
				onDisconnected={() => alert("disconnected")}
			/>
		</div>
	);
}
```

### Register Data

- Add [`RegisterData.tsx`](./src/RegisterData.tsx) Component

```tsx
import { detectConcordiumProvider } from "@concordium/browser-wallet-api-helpers";
import {
	AccountTransactionType,
	DataBlob,
	RegisterDataPayload,
	sha256,
} from "@concordium/web-sdk";
import { Button, Link, Stack, TextField, Typography } from "@mui/material";
import { FormEvent, useState } from "react";
import { Buffer } from "buffer/";

export default function RegisterData() {
	let [state, setState] = useState({
		checking: false,
		error: "",
		hash: "",
	});

	const submit = async (event: FormEvent<HTMLFormElement>) => {
		event.preventDefault();
		setState({ ...state, error: "", checking: true, hash: "" });
		const formData = new FormData(event.currentTarget);

		var formValues = {
			data: formData.get("data")?.toString() ?? "",
		};

		if (!formValues.data) {
			setState({ ...state, error: "Invalid Data" });
			return;
		}

		const provider = await detectConcordiumProvider();
		const account = await provider.connect();

		if (!account) {
			alert("Please connect");
		}

		try {
			const txnHash = await provider.sendTransaction(
				account!,
				AccountTransactionType.RegisterData,
				{
					data: new DataBlob(sha256([Buffer.from(formValues.data)])),
				} as RegisterDataPayload
			);

			setState({ checking: false, error: "", hash: txnHash });
		} catch (error: any) {
			setState({ checking: false, error: error.message || error, hash: "" });
		}
	};

	return (
		<Stack
			component={"form"}
			spacing={2}
			onSubmit={submit}
			autoComplete={"true"}
		>
			<TextField
				id="data"
				name="data"
				label="Data"
				variant="standard"
				disabled={state.checking}
			/>
			{state.error && (
				<Typography component="div" color="error">
					{state.error}
				</Typography>
			)}
			{state.checking && <Typography component="div">Checking..</Typography>}
			{state.hash && (
				<Link
					href={`https://dashboard.testnet.concordium.com/lookup/${state.hash}`}
					target="_blank"
				>
					View Transaction <br />
					{state.hash}
				</Link>
			)}
			<Button
				type="submit"
				variant="contained"
				fullWidth
				size="large"
				disabled={state.checking}
			>
				Register Data
			</Button>
		</Stack>
	);
}
```

- Update [`App.tsx`](./src/App.tsx) with the newly added component

```tsx
import "./App.css";
import Header from "./Header";
import { useState } from "react";
import { Container } from "@mui/material";
import RegisterData from "./RegisterData";

export default function App() {
	const [isConnected, setConnected] = useState(false);

	return (
		<div className="App">
			<Header
				onConnected={() => setConnected(true)}
				onDisconnected={() => setConnected(false)}
			/>
			<Container sx={{ mt: 5 }}>{isConnected && <RegisterData />}</Container>
		</div>
	);
}
```
