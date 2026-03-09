# Proyect
use anchor_lang::prelude::*;

declare_id!("11111111111111111111111111111111");

#[program]
pub mod votacion {
    use super::*;

    pub fn crear_encuesta(ctx: Context<CrearEncuesta>, pregunta: String) -> Result<()> {
        let encuesta = &mut ctx.accounts.encuesta;
        encuesta.pregunta = pregunta;
        encuesta.votos_si = 0;
        encuesta.votos_no = 0;
        Ok(())
    }

    pub fn votar_si(ctx: Context<Votar>) -> Result<()> {
        let encuesta = &mut ctx.accounts.encuesta;
        encuesta.votos_si += 1;
        Ok(())
    }

    pub fn votar_no(ctx: Context<Votar>) -> Result<()> {
        let encuesta = &mut ctx.accounts.encuesta;
        encuesta.votos_no += 1;
        Ok(())
    }
}

#[account]
pub struct Encuesta {
    pub pregunta: String,
    pub votos_si: u64,
    pub votos_no: u64,
}

#[derive(Accounts)]
pub struct CrearEncuesta<'info> {
    #[account(init, payer = usuario, space = 8 + 200)]
    pub encuesta: Account<'info, Encuesta>,
    #[account(mut)]
    pub usuario: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Votar<'info> {
    #[account(mut)]
    pub encuesta: Account<'info, Encuesta>,
}import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { expect } from "chai";

describe("votacion", () => {

  const provider = anchor.AnchorProvider.env();
  anchor.setProvider(provider);

  const program = anchor.workspace.Votacion;

  const encuesta = anchor.web3.Keypair.generate();

  it("Crea una encuesta", async () => {

    await program.methods
      .crearEncuesta("¿Te gusta Solana?")
      .accounts({
        encuesta: encuesta.publicKey,
        usuario: provider.wallet.publicKey,
        systemProgram: anchor.web3.SystemProgram.programId,
      })
      .signers([encuesta])
      .rpc();

    const cuenta = await program.account.encuesta.fetch(encuesta.publicKey);

    expect(cuenta.votosSi.toNumber()).to.equal(0);
    expect(cuenta.votosNo.toNumber()).to.equal(0);

  });

  it("Registra voto SI", async () => {

    await program.methods
      .votarSi()
      .accounts({
        encuesta: encuesta.publicKey,
      })
      .rpc();

    const cuenta = await program.account.encuesta.fetch(encuesta.publicKey);

    expect(cuenta.votosSi.toNumber()).to.equal(1);

  });

  it("Registra voto NO", async () => {

    await program.methods
      .votarNo()
      .accounts({
        encuesta: encuesta.publicKey,
      })
      .rpc();

    const cuenta = await program.account.encuesta.fetch(encuesta.publicKey);

    expect(cuenta.votosNo.toNumber()).to.equal(1);

  });

});import * as anchor from "@coral-xyz/anchor";

const provider = anchor.AnchorProvider.env();
anchor.setProvider(provider);

const program = anchor.workspace.Votacion;

const encuesta = anchor.web3.Keypair.generate();

async function main() {

  console.log("Creando encuesta...");

  await program.methods
    .crearEncuesta("¿Te gusta Solana?")
    .accounts({
      encuesta: encuesta.publicKey,
      usuario: provider.wallet.publicKey,
      systemProgram: anchor.web3.SystemProgram.programId,
    })
    .signers([encuesta])
    .rpc();

  console.log("Encuesta creada:", encuesta.publicKey.toString());

  let cuenta = await program.account.encuesta.fetch(encuesta.publicKey);

  console.log("Pregunta:", cuenta.pregunta);
  console.log("Votos SI:", cuenta.votosSi.toNumber());
  console.log("Votos NO:", cuenta.votosNo.toNumber());

  console.log("Votando SI...");

  await program.methods
    .votarSi()
    .accounts({
      encuesta: encuesta.publicKey,
    })
    .rpc();

  cuenta = await program.account.encuesta.fetch(encuesta.publicKey);

  console.log("Votos SI ahora:", cuenta.votosSi.toNumber());

  console.log("Votando NO...");

  await program.methods
    .votarNo()
    .accounts({
      encuesta: encuesta.publicKey,
    })
    .rpc();

  cuenta = await program.account.encuesta.fetch(encuesta.publicKey);

  console.log("Votos NO ahora:", cuenta.votosNo.toNumber());
}

main();