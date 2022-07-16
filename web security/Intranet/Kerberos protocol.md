# Kerberos protocol

The Kerberos protocol is a computer network authentication protocol, which is mainly used in Windows 2000 server (or later) domain environment.

## Composition role

Client: the user.

Server: the service provider.

**KDC: the key distribution center**, who relies on two additional administrative agents.

- **KAS: the Kerberos authentication server**, who authenticates the user and provides the corresponding client with **ticket granting tickets (TGT)*- to use the network for the day.

- **TGS: the ticket granting server**, who authenticates the client to each requested server based on the TGT and provides the corresponding client with **service tickets (ST)*- to access the requested service.

## Authentication exchange

The whole authentication process is mainly divided into six steps:

$$
    \begin{aligned}
    KRB\_AS\_REQ \quad &1. \ C \longrightarrow K: \ C, T, n_1\\
    KRB\_AS\_REP \quad &2. \ K \longrightarrow C: \ C, \underbrace{{\{AK, C, t_K\}}_{k_T}}_{TGT}, {\{AK, n_1, t_K, T\}}_{k_C}\\
    KRB\_TGS\_REQ \quad &3. \ C \longrightarrow T: \ \overbrace{{\{AK, C, t_K\}}_{k_T}}^{TGT}, {\{C, t_C\}}_{AK}, S, n_2\\
    KRB\_TGS\_REP \quad &3. \ T \longrightarrow C: \ C, \underbrace{{\{SK, C, t_T\}}_{k_S}}_{ST}, {\{SK, n_2, t_T, S\}}_{AK}\\
    KRB\_AP\_REQ \quad &4. \ C \longrightarrow S: \ \overbrace{{\{SK, C, t_T\}}_{k_S}}^{ST}, {\{C, t^{'}_C\}}_{SK}\\
    KRB\_AP\_REP \quad &5. \ S \longrightarrow C: \ {\{t^{'}_C\}}_{SK}
    \end{aligned}
$$

We will now describe each of the three roundtrips between a client (C) and the KAS (K for short), the TGS (T for short), and a server (S), respectively.

### Authentication service (AS) exchange ($C ⇔ K$)

The client process $C$ generates a nonce $n_1$ and sends it to the KAS together with her own name, $C$, which
indirectly identifies the user, and the name of the TGS (officially “krbtgt”, here abbreviated as $T$).

Upon recognizing $C$,  the KAS replies with a message containing two encrypted components:

- The TGT ${\{AK, C, t_K\}}_{k_T}$ that is cached by $C$ and will be used to obtain STs for the rest of the day. The TGT is meant for the TGS and is encrypted with the long-term key $k_T$ that the KAS shares with the TGS. It contains a freshly generated authentication key $AK$ and a timestamp $t_K$ in addition to $C$’s name.

- ${\{AK, n_1, t_K, T\}}_{k_C}$ with which the KAS informs $C$ of the parameters of the ticket.  The key $k_C$ used to encrypt the second component is a long-term secret between $C$ and the KAS derived from the user’s password. $AK$ will be used in every subsequent communication with the TGS, reducing the use of $C$'s long-term key $k_C$.

The timestamp $t_K$ will assure the TGS and C that this ticket was issued recently, as all Kerberos principals have loosely synchronized clocks. The nonce $n_1$ in the second component binds this response to $C$'s original request.

### Ticket granting (TG) exchange ($C ⇔ T$)

In the outgoing message, $C$ transmits the cached TGT and $S$'s name together with a freshly generated nonce $n_2$ (again to bind this request and the subsequent response), and the authenticator ${\{C, t_C\}}_{AK}$, where $t_C$ is a timestamp. The authenticator proves to $T$ that $C$ indeed knows the authentication key $AK$.

Upon authenticating $C$ and verifying that she is allowed to use $S$, the TGS sends a response containing two encrypted components:

- the ST ${\{SK, C, t_T\}}_{k_S}$ is encrypted with the long-term key $k_S$ shared between the KDC and $S$, and it contains a freshly generated service key $SK$, $C$'s name, and a timestamp $t_T$.

- ${\{SK, n_2, t_T, S\}}_{AK}$. The other encrypted component is as in the second message above, but now encrypted with the authentication key $AK$. $C$ caches the $ST$.

### Client/server exchange ($C ⇔ S$)

With a ST in hand, $C$ simply contacts $S$ with this ticket and an authenticator similar to the one described above.

The response from $S$ is optional as the subsequent application exchanges may subsume it. When present, it provides assurance to $C$ that $S$ is alive, for example by returning the timestamp $t^{'}_C$ that $C$ included in her request, encrypted with the service key.
