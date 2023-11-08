---
title: Liquidity Advertisements - niftynei - TABConf 2021
transcript_by: natan-del-prado via review.btctranscripts.com
media: https://www.youtube.com/watch?v=yXemwW73W5Q
tags: ["Lightning"]
speakers: ["Lisa Neigut"]
categories: ["conference"]
date: 2021-11-06
---

## Introduction

Again, my name is Lisa Neigut. I work at Blockstream. I'm here today to talk to you about the spec proposal that we've added to c-lightning and hoping to get into more Lightning implementation soon to make it standard. This is about liquidity advertisements. So today I'm going to be talking to you about what a liquidity ad is, how they work, kind of explain how data gets passed around Lightning Network just in general, and maybe get some of you excited about using them.

## What is a Liquidity Ad?

So let's get started. So what is a liquidity ad? Okay, at a high level, a liquidity ad lets you tell other nodes that you'll put funds into a Lightning channel when they open one to you and that you want them to pay you some money when you do this. So let's just kind of like walk through how channel opening works and maybe this will give you a good understanding of like what role liquidity ads play in this process. So let's say you're a Lightning node. Let's call you Harry. So this is Harry, the Lightning node. So normally when you open a Lightning channel, just one side, like Harry in this case, will commit funds to that channel. So he'll take his on-chain funds and he'll put them into a channel and he's making a channel with another node in this case called Heart. So Harry is opening a channel to Heart and he's got on-chain funds that he's put into the channel. So this means that Harry can send a payment out over this channel since he has funds on his side, but immediately as soon as the channel's open, he can't receive any payments incoming from Heart's side because there's no capital or money that's been put into the channel on Heart's side of the channel. But what if Harry could coordinate with Heart? So when he opened the channel, she would also put funds into the channel on her side. That way when they open the channel, there would be funds on both sides of the channel and Harry could immediately both send and receive through the same channel using funds that both he and Heart had put into the channel. So the problem is that Harry and Heart need to coordinate, right? Heart doesn't know that Harry's going to open a channel. She doesn't know how much money he wants her to put into the channel for him. So maybe there's some way that Harry can call up Heart and ask her to put funds in the channel for him, like telephones or contacting her over telegram or something like that. But this takes a lot of coordination. You need to be able to find the person who runs the Heart node on the network. If you've looked at nodes on the network before, they usually just have aliases and pub keys, but it's kind of hard to figure out exactly who to contact if you want to open a channel with a particular node. So if Harry doesn't have Heart's phone number, for example, this isn't going to work, right? So instead, with liquidity ads, this kind of lets Heart put up an advertisement advertising that she has liquidity that she's willing to put into channels. And she tells people what her rates are, how much she wants in order to put this capital into a channel on their behalf. So Harry can see that Heart is interested in putting money in a channel with him and how much she wants in return for that capital. So Heart puts up a little advertisement. This is called the liquidity ad part. That's why you put advertisement in the name. And what if, in fact, it's not just Heart that can put up an ad, but it's anyone who's running a node on the Lightning Network can put up an advertisement. So now Harry has a lot of options about what nodes he can get inbound liquidity from. So he can pick what node out of all of them to ask to put funds in a channel with him. Or maybe he opens channels with a lot of nodes and has them put Bitcoin on their side of the channel at start. So he can immediately start receiving payments from all of these nodes across the Lightning Network. So from a high level, that's kind of the goal that we're trying to achieve with liquidity networks or like what we're trying to accomplish.

## How Does a Node Put Up a Liquidity Ad?

So that's kind of what we're trying to do. Let's talk a little bit of how these work. So from a high level, a liquidity ad lets you tell other nodes that you'll put funds into Lightning Channel when they open one to you. And it also tells you the rates that they want you to pay them for that capital. So here's how the liquidity ads get passed around the network. It's not like in Lightning you can just put up a little sign on your node. That's not exactly how computers work. Instead, we use something called the Gossip Network. And I will explain very quickly how the Gossip Network works in Lightning. So again, we're back with Harry. Harry wants to let other nodes on the Lightning Network know things about himself. So he builds this message. This particular message is called the node announcement. It contains information about Harry's node. This message is part of the Lightning spec. And in fact, in a minute, provided I can get all of the AV stuff to work correctly, I will show you what it looks like in the spec. That'll be fun. So he creates this message called the node announcement. And then he sends it out to all the peers that he's currently connected to. So he sends this message out. And then those peers get his message. They do a little bit of validation to make sure it's a valid node announcement. I won't talk too much today about what the requirements are for valid gossip messages. But just know that there's a little bit of validation that goes on when a node receives one. Then once they've validated that message, they will go ahead and pass it on to their peers in turn. So this way, every node who's connected to other nodes on the Lightning Network receives information about Harry's node in the node announcement.

## What is in a Node Announcements?

Okay, so what kind of information is in a node announcement? This is kind of just interesting stuff to know. One thing that we put in the node announcement is an alias or like your node's name. So maybe Harry would call his node Harry's node. That's not the best security. People could see that and then figure out you're Harry. And they're like, oh, Harry, there's a node on Lightning called Harry's node. So maybe that's not the best name for a node. But yeah, so this is how if you ever look at a Lightning Network Explorer and they have names, those names come from the node announcements that every node is sending out to all the other nodes on the network. More information, what other information is in these? You can put a color so you can like color code your node. So when people make graphs of the Lightning Network, when nodes have colors, those are colors that typically they've picked and put in their node announcement and then told everyone about. There's also a list of addresses that you can use to reach Harry. So this is kind of like his phone address listings. This is how you open a channel with Harry or create a new connection with him. So to list. Right now we can list IP addresses and you can list Tor addresses. We've got some spec proposals out to add more address types to this. You can maybe put like a DNS in here or like something else. But yeah, we include a list of features that this node supports. So for example, like we have this special kind of gossip queries thing. So only certain nodes support it. So we would flag that we support this particular feature in this node announcement. And then there's a signature. This makes sure that Heart can't send out a message and pretend to be Harry. Only Harry can send out a message and people can verify that it is in fact a message that Harry sent because Harry signed it. So signatures are an important part of these messages. And then it includes a timestamp. This is to kind of help figure out what or if Harry decides to update his node announcement, he'll send out an update. The timestamp helps you figure out what the most recent one is so you can delete older versions of this message that you've received. Cool. Okay. So the way the liquidity ad works is we take this existing node announcement and we've added a little bit of extra data to it. So you kind of have this like existing message and then there's an advertisement that's like added a little bit more. Here's the information a liquidity ad adds to the existing node announcement.

## What Liquidity Ads Add to Node Announcements?

The first thing that we add is like how much. So like Heart, for example, is going to add this to her liquidity. I'm sorry. Heart's going to add this to her node announcement. The first thing she's going to figure out is, okay, let's say Heart has, I don't know, a million sats that she wants to offer up to provide liquidity to other nodes. So people can pay her for this million sats to put into a channel so they can receive payments through it. First thing she's going to decide is how much she wants to charge for that service of putting that liquidity into a channel with another node. So in liquidity ads, there's two different ways she can do that or maybe both. She could put a flat fee so she could be like, okay, I want a thousand sats. And if you ask for 500,000 sats or a million sats, you're just going to pay me 500 sats for that money that I put into the channel. She can also add like a percentage, which we call like a basis rate. This is like so if she had just a basis rate and you ask for a million sats, you would pay a percentage of that million, like maybe 1% or half a percent. Whereas if you only ask for 500,000 sats, you could pay her like a little bit less because you're asking for less of her total liquidity that's available. So those are the first two fields that we add to that you have in liquidity ad. The next thing that we have is kind of like an interesting, this kind of has to do with what it costs to open a channel. So when you're opening a new Lightning channel, that requires making an on-chain transaction. With liquidity ads, Heart is going to be putting money into a channel. In Bitcoin, in order to put money into an on-chain transaction, that means that there's going to be inputs that Heart has, like UTXOs that Heart is spending on this transaction. Anytime that you include an input on a transaction, there's a weight, like a cost associated with that. You have to pay on-chain fees for any data that you put on an on-chain transaction. So because Heart is doing a favor to Harry by putting this money, by spending these UTXOs in a transaction for him, we're asking Harry to help cover the on-chain cost of how many bytes that's going to be. So basically, Heart in her liquidity advertisement will put up how many bytes of on-chain data she's going to ask Harry to cover for her. So that when she adds money to this on-chain transaction for him, Harry is actually paying for the cost of that data to appear on the chain. So it's kind of cool, right? So Heart doesn't actually have to pay to get those UTXOs deployed on-chain. Harry is going to reimburse her, basically, for the cost of adding that data to the chain so that when the channel opens, Harry has inbound capacity, right? So it's in Harry's favor that Heart does this. So as part of the negotiation, Heart can tell Harry how many bytes of data she wants him to pay her for this on-chain transaction. And it's kind of complicated, but it's cool. If you ever see these, if you see a liquidity ad, it'll just be a number that's in weight. Maybe we'll go through one here in a second so you can see what they look like. And then the final thing that we do is kind of an interesting sort of detail. So whenever you put money into a Lightning channel and someone routes a payment through it, you get paid routing fees, right? And that's the person who gets paid that is the one whose capital moved from their side of the channel to the other side. Heart could get Harry to pay her to put money into a channel and then charge really high fees for anyone to use that capital that she just put into a channel. And if she did that, then for Harry, that's not very useful, right? So if Heart puts like a million sats into her side of the channel, but charges like a 5% fee in order to move that capital to Harry's side, Harry is probably not going to get, whoever's going to be using that inbound capacity towards Harry, like paying Harry is going to be paying a lot to pay Harry, and Heart's going to be benefiting from that. So as part of the liquidity ads, what we do is we add a kind of promise that Heart makes to Harry that the maximum amount that they'll charge for someone to route through those funds. So the maximum fee that someone would pay to send Harry a payment through Heart's channel, Heart's going to commit to that in the advertisement. So what that means is when you look at all the advertisements, you can kind of see who's charging, who's got like what maximum fee rates that they're charging for the capital they put in their channels. She could always charge less than that, but she kind of commits to a cap. And that lets you, when you're looking at all these advertisements, kind of figure out if someone is going to attempt to scalp the people who are sending payments through you or not, basically. So it's information that you can use to compare different liquidity ads when you see all of them. So just to recap, there's really kind of three things that are in a liquidity ad. One of them is how much you want to be paid for your capital that you're putting in the channel. The other is kind of like how much of on-chain fees the person who opens it will be giving to the person who opened it. And then the final thing is a promise about how much you'll have to pay in order to use those funds that you've allocated to the channel. Cool. Okay, cool. Oh, and then here's my little illustration of where channel fees happen. So if a payment is coming from, a payment is flowing, so Heart has committed capital to this channel, and now Harry is receiving a payment through Heart's side of the channel. So it's coming from someone else into Harry's side of the channel, so Harry is receiving money. The little bit of money, there's going to be a little bit of money in that payment that gets dropped off at Heart's node. And that's basically as a favor for Heart pushing the money over to Harry, and that little money that Heart earns on that push is the channel fee. And that's how channel fees work across the entire Lightning Network. Cool. Okay, cool.

## Recap

So just to recap really quickly, a liquidity ad is some extra data on your node announcement, and it states the rates that you're charging for capital, some on-chain fees, and a guarantee to the max you'll charge anyone to use that capital that you've put into a channel. So why should you be excited about liquidity ads? It's a decentralized way to find inbound liquidity, so if you're starting up a node and you want to get some money flowing towards you, this is a really easy way to look out and see who's got capital and who wants to allocate it and what they're asking you for in return for that. And it's easy to see who in the network is offering them up because it's really lightweight, it just ends up in gossip, because everyone in the network just kind of puts them up and you can go through and see them. There's also this project by a guy named Severin out of somewhere in Europe called Ellen Router. He's got this lookups thing, he's adding a GUI. This hasn't been launched yet, but hopefully this will be out sometime this month. He's added a UI, so when you look through all of the different nodes that are offering liquidity ads, you'll be able to figure out how much it would cost to get them to put money into the channel for you, like a nice little slider, etc. I can play that again. Sorry. Okay, I don't know. Yeah, anyway, so it's like a nice little slider and it has all the data we just talked about really quickly. Cool. What's the other thing? The other nice thing about liquidity ads is it lets you do your own research, so you can see exactly who in the network is offering it up, and so you can figure out what their inbound and outbound capacity looks like, and so you can figure out what any of their liquidity would be worth to you, and decide whether or not what they're asking you to pay is worth it. And then finally, it's easy for anyone to participate. All you have to do is add some configs to your node and start advertising that you have liquidity. Cool. So if you are running c-lightning, or if you want to run c-lightning to run a liquidity ad, there's an article here at this QR code that will let you look up, I don't know, it tells you how to set these up on c-lightning and get running, etc. Cool. And then, yeah, and then c-lightning, we've recently started a Discord, so that's a project I work on. If you want to hang out and talk about liquidity ads or c-lightning plugins or anything in general, we would love to have you come join the conversation.

## Q&A

Cool. I think I do have a few minutes for questions. I think I saw a hand if anyone else wants. Yeah. Okay. So the question is, what does PPM stand for? That is a Lightning Network kind of, stands for, so the PPM is parts per million. It's kind of like a basis rate except, like, so like if you're used to, it's like a percentage. So how much you would pay in fees for every million millisatoshis that goes through a channel. So one PPM means you would pay one millisatoshi for every million millisatoshis that you're routing, for example. So it's basically just like a rate of percent, well, it's like even more granular than a percent of how much you pay when you do a channel fee. But that's a channel fee. Those are usually used for channel fees when you're figuring out how much a channel fee is. Yeah. The question is, can the fee rate that Heart promises be set to range? Not currently. Not in the current spec. Yeah. Not currently. Right now it's like you just set your fee. Yes. Yeah, so there's a limit on how often the node announcements can be updated. It's every five minutes. And that's set by the spec. So c-lightning won't let you violate that. Like if you said, if you updated it, the update won't go out until five minutes later. And I think I'm out of time, but just a note on the Gossip Network in general, there's a couple other messages that get sent around the Gossip Network. This is also how you advertise what your channel fees are. If you update your channel fees, it usually takes an hour or more for them to get propagated across the network. So even if you update it every five minutes, it might take an hour plus for people to actually start seeing those changes, depending on where they are. Cool. Okay, great. Well, I hope that you guys enjoyed this talk. And if you want to learn more about channel leasing and liquidity stuff, I'm doing a talk with Ryan Gentry later this afternoon over in the Socratic Village. And I look forward to talking about this more. Great. Thank you.