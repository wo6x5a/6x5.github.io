---
title: spring data jpa 查询使用案例
date: 2017-03-27 17:29:11
tags: [java,spring data jpa]
categories: 技术
---

```java
/**
 * 
 */
public String getSumAmount(final AcctDtlSearchDto searchDto, final ERecvPayFlag recvPayFlag) {  
	String startDate = searchDto.getStartDate();                                                
	String endDate = searchDto.getEndDate();                                                    
	EUseType useType = searchDto.getUseType();                                                  
	CriteriaBuilder builder = em.getCriteriaBuilder();                                          
	CriteriaQuery<BigDecimal> query = builder.createQuery(BigDecimal.class);                    
	Root<SubAcctTrxJnlPo> root = query.from(SubAcctTrxJnlPo.class);                             
	Predicate predicate = builder.conjunction();                                                
	List<Expression<Boolean>> expressions = predicate.getExpressions();                         
	Expression<BigDecimal> sumTrxAmt = builder.sum(root.<BigDecimal> get("trxAmt"));            
	if (StringUtils.isNotBlank(startDate)) {                                                    
		expressions.add(builder.greaterThanOrEqualTo(root.<String> get("trxDate"), startDate)); 
	}                                                                                           
	if (StringUtils.isNotBlank(endDate)) {                                                      
		expressions.add(builder.lessThanOrEqualTo(root.<String> get("trxDate"), endDate));      
	}                                                                                           
	if (EUseType.ALL != useType) {                                                              
		expressions.add(builder.equal(root.<EUseType> get("useType"), useType));                
	}                                                                                           
	if (null != recvPayFlag) {                                                                  
		expressions.add(builder.equal(root.<ERecvPayFlag> get("recvPayFlag"), recvPayFlag));    
	}                                                                                           
	query.where(predicate);                                                                     
	query.select(sumTrxAmt);                                                                    
	BigDecimal result = em.createQuery(query).getSingleResult();                                
	result = (null == result ? BigDecimal.ZERO : result);                                       
	return result.toString();                                                                   
}                   
```
<!--more-->
```java
/**
 * 
 */
private BigDecimal getWarrantedProjectAmt(final String userId) {
	List<ProjectPo> list = projectRepository.findAll(new Specification<ProjectPo>() {
	@Override
	public Predicate toPredicate(Root<ProjectPo> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
		Predicate predicate = cb.conjunction();
		List<Expression<Boolean>> expressions = predicate.getExpressions();
		expressions.add(cb.equal(root.<String> get("wrtrId"), userId));
		List<EProjStatus> inStatus = Arrays.asList(EProjStatus.RISK_AUDIT, EProjStatus.RISK_RATING);
		expressions.add(root.<EProjStatus> get("status").in(inStatus));
		return predicate;
		}
	});
	BigDecimal warrantAmt = BigDecimal.ZERO;
	for (ProjectPo proj : list) {
		warrantAmt = warrantAmt.add(proj.getProjectAmt());
	}
	return warrantAmt;
}



/**
 * 
 */
public List<ProdServItemDto> getItems(final ProdServItemSearchDto searchDto, final String currUserId){
	long times = System.currentTimeMillis();
	List<ProdServInfoPo> list = prodServInfoRepo.findAll(new Specification<ProdServInfoPo>(){
		@Override
		public Predicate toPredicate(Root<ProdServInfoPo> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
			Predicate predicate = cb.conjunction();
			String workDate = CommonBusinessUtil.getCurrentWorkDate();
                List<Expression<Boolean>> expressions = predicate.getExpressions();
                // 已授信过滤
                expressions.add(cb.exists(getProdServParamSubQuery(query, cb, workDate, root.<String>get("userId"))));
                if(StringUtils.isNotBlank(searchDto.getName())){
                	expressions.add(cb.like(root.<EnterpriseInfoPo>get("enterpriseInfoPo").<String>get("name"), "%"+searchDto.getName()+"%"));
                }
                if(StringUtils.isNotBlank(searchDto.getTelephone())){
                	expressions.add(cb.like(root.<EnterpriseInfoPo>get("enterpriseInfoPo").<String>get("telephone"), "%"+searchDto.getTelephone()+"%"));
                }
                Expression<Boolean> servProvTypes = root.<EWarrantProdServProvType> get("servProvType").in(EWarrantProdServProvType.MICRO_CMPY, EWarrantProdServProvType.WRTR_CMPY);
                Expression<Boolean> unionServProvType = cb.and(
                		cb.equal(root.<EWarrantProdServProvType> get("servProvType"), EWarrantProdServProvType.UNION_CMPY), 
                		cb.exists(getSubQuery(query, cb, currUserId, root.<String> get("userId"))));
                expressions.add(cb.or(servProvTypes, unionServProvType));
                /** 如果是小贷,不显示自己 **/
                expressions.add(cb.notEqual(root.<String>get("userId"), currUserId));
                return predicate;
			}
			private Subquery<String> getSubQuery(CriteriaQuery<?> query, 
					CriteriaBuilder cb, String fncrId, Path<String> prodServUserId) {
                Subquery<String> subQuery = query.subquery(String.class);
                Root<ProdServInfoPo> prodServ = subQuery.from(ProdServInfoPo.class);
                Join<ProdServInfoPo, ProdServGroupPo> prodServGroup = prodServ.join("prodServGroupList", JoinType.INNER);
                Join<ProdServGroupPo, GroupPo> group = prodServGroup.join("groupPo", JoinType.INNER);
                Join<GroupPo, GroupUserPo> groupUser = group.join("groupUserPoSet", JoinType.INNER);
                Join<GroupUserPo, FncrInfoPo> fncrInfo = groupUser.join("fncrInfoPo", JoinType.INNER);
                return subQuery.select(prodServ.<String>get("userId")).where(
                		cb.and(cb.equal(prodServ.get("userId"), prodServUserId), 
                				cb.equal(fncrInfo.<String> get("userId"), fncrId)));
            }
			
			private Subquery<String> getProdServParamSubQuery(CriteriaQuery<?> query, 
					CriteriaBuilder cb, String workDate, Path<String> userId) {
                Subquery<String> subQuery = query.subquery(String.class);
                Root<ProdServParamPo> param = subQuery.from(ProdServParamPo.class);
                return subQuery.select(param.<String>get("userId")).where(
                		cb.and(cb.equal(param.get("userId"), userId), 
                				cb.lessThanOrEqualTo(param.<String>get("startDate"), workDate), 
                				cb.greaterThanOrEqualTo(param.<String>get("endDate"), workDate)
                		));
			}
		});
		List<ProdServItemDto> items = new ArrayList<ProdServItemDto>();
		for(ProdServInfoPo serv: list){
			EnterpriseInfoPo ent = serv.getEnterpriseInfoPo();
			ProdServItemDto item = new ProdServItemDto();
			item.setUserId(serv.getUserId());
			item.setWrtrKind(serv.getWrtrKind());
			item.setServProvType(serv.getServProvType());
			item.setQq(ent.getQq());
			item.setName(ent.getName());
			item.setAddress(ent.getAddress());
			item.setZipCode(ent.getZipCode());
			item.setContractName(ent.getContractName());
			item.setTelephone(ent.getTelephone());
			item.setWrtrFileId(serv.getWrtrFileId());
			items.add(item);
		}
		logger.debug("cost times is {} ms", (System.currentTimeMillis()-times));
		return items;
	}
	
//expressions.add(cb.greaterThanOrEqualTo(root.<Date> get("createTs"), DateUtils.getStartDate(DateUtils.getDate(fromDate, YYYY_MM_DD))));
//expressions.add(cb.lessThanOrEqualTo(root.<Date> get("createTs"), DateUtils.getEndDate(DateUtils.getDate(toDate, YYYY_MM_DD))));

/**
*公告查询,这里主要看根据roleIds 不同用户显示不同的公告
*/
public DataTablesResponseDto<ArticleDto> getArticlesForHome(final ArticleSearchDto searchCriteria) {
Pageable pageable = PaginationUtil.buildPageable(searchCriteria,
		PersistenceUtil.getIdName(ArticlePo.class));
Specification<ArticlePo> specification = new Specification<ArticlePo>() {
	@Override
	public Predicate toPredicate(Root<ArticlePo> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
		Predicate predicate = cb.conjunction();
		List<Expression<Boolean>> expressions = predicate.getExpressions();
		if (null != searchCriteria) {
			String nowDateStr = DateUtils.formatDate(new Date(), ApplicationConsts.DATE_FORMAT);
			EArticleStatus status = searchCriteria.getStatus();
			EColumnCategory categoryId = searchCriteria.getCategoryId();
			List<Predicate> predicateList = new ArrayList<Predicate>();
			List<UserRolePo> userRolePoList = userRoleRepository.findByUserId(securityContext
					.getCurrentUserId());

			for (UserRolePo userRole : userRolePoList) {
				predicateList.add(cb.like(root.<String> get("roleIds"), "%" + userRole.getRoleId()
						+ "%"));
			}
			predicateList.add(cb.isNull(root.<String> get("roleIds")));

			Predicate[] predicate1 = new Predicate[predicateList.size()];
			Predicate predicate2 = cb.or(predicateList.toArray(predicate1));
			expressions.add(predicate2);

			expressions.add(cb.lessThanOrEqualTo(root.<String> get("startDt"), nowDateStr));
			expressions.add(cb.greaterThanOrEqualTo(root.<String> get("endDt"), nowDateStr));
			if (EArticleStatus.ALL != status && null != status) {
				expressions.add(cb.equal(root.<EArticleStatus> get("status"), status));
			}
			if (EColumnCategory.ALL != categoryId && null != categoryId) {
				expressions.add(cb.equal(root.<EColumnCategory> get("categoryId"), categoryId));
			}
		}
		return predicate;
	}
};
Page<ArticlePo> articlePos = articleRepository.findAll(specification, pageable);
DataTablesResponseDto<ArticleDto> result = PaginationUtil.populateFromPage(articlePos,
		new Function<ArticlePo, ArticleDto>() {
			@Override
			public ArticleDto apply(ArticlePo po) {
				ArticleDto articleDto = ConverterService.convert(po, ArticleDto.class);
				return articleDto;
			}
		});
return result;
}



/**
 * 
 */
public BigDecimal getSumInvstAmt(String acctNo, String productId){
	CriteriaBuilder cb = em.getCriteriaBuilder();
	CriteriaQuery<BigDecimal> query = cb.createQuery(BigDecimal.class);
	Root<ProductSubscribeDtlPo> root = query.from(ProductSubscribeDtlPo.class);
	Expression<BigDecimal> subsAmtCm = cb.prod(root.<BigDecimal>get("unitAmt"), root.<Long>get("unit").as(BigDecimal.class));
	Expression<BigDecimal> sumSubAmtQuery = cb.sum(subsAmtCm);
	Predicate predicate = cb.conjunction();
	List<Expression<Boolean>> expressions = predicate.getExpressions();
	if(StringUtils.isNotBlank(acctNo)){
		expressions.add(cb.equal(root.<String>get("acctNo"), acctNo));
	}
	if(StringUtils.isNotBlank(productId)){
		expressions.add(cb.equal(root.<String>get("productId"), productId));
	}
	query.select(sumSubAmtQuery);
	query.where(predicate);
	BigDecimal result = em.createQuery(query).getSingleResult();
	return result==null?BigDecimal.ZERO:result;
}


/**
 * 
 */
private List<ProjectPo> getProjectByFncrId(final String fncrAcctNo) {
	List<ProjectPo> list = projectRepository.findAll(new Specification<ProjectPo>() {
		@Override
		public Predicate toPredicate(Root<ProjectPo> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
			Predicate predicate = cb.conjunction();
			List<Expression<Boolean>> expressions = predicate.getExpressions();
			expressions.add(cb.equal(root.<String> get("fncrAcctNo"), fncrAcctNo));
			List<EProjStatus> inStatus = Arrays.asList(EProjStatus.INIT, EProjStatus.WRTR_AUDIT,
					EProjStatus.RISK_AUDIT, EProjStatus.RISK_RATING, EProjStatus.PASSED);
			expressions.add(root.<EProjStatus> get("status").in(inStatus));
			return predicate;
		}
	});
	return list;
}


/**
 * 
 */
@Transactional(readOnly = true)
public List<SimpleMemberDto> searchFncrMembers(final String keyWord) {
	Sort sort = new Sort(Direction.ASC, PersistenceUtil.getIdName(AcctPo.class));
	Pageable pageable = new PageRequest(0, 20, sort);
	Page<AcctPo> page = acctRepository.findAll(new Specification<AcctPo>() {
		@Override
		public Predicate toPredicate(Root<AcctPo> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
			Predicate predicate = cb.conjunction();
			List<Expression<Boolean>> expressions = predicate.getExpressions();
			expressions.add(cb.exists(getSubQuery(query, cb, root.<String>get("userId"))));
			String keyWordTrim = keyWord.trim();
			if (StringUtils.isNotBlank(keyWord)) {
				expressions.add(cb.or(
						cb.like(root.<String> get("acctNo"), "%" + keyWordTrim + "%"),
						cb.or(cb.like(root.<UserPo> get("userPo").<String> get("sname"), "%" + keyWordTrim
								+ "%"),
								cb.like(root.<UserPo> get("userPo").<String> get("mobile"), "%" + keyWordTrim
										+ "%"))));
			} else {
				expressions.add(cb.equal(root.<String> get("acctNo"), "0"));
			}
			return predicate;
		}
		
		private Subquery<String> getSubQuery(CriteriaQuery<?> query, 
				CriteriaBuilder cb, Path<String> userId) {
			Subquery<String> subQuery = query.subquery(String.class);
			Root<SimpleUserPo> suser = subQuery.from(SimpleUserPo.class);
			Join<SimpleUserPo, FncrInfoPo> fncrInfo = suser.join("fncrInfoPo", JoinType.INNER);
			return subQuery.select(suser.<String>get("userId")).where(
					cb.and(cb.equal(suser.get("userId"), userId), 
							cb.equal(suser.get("userId"), fncrInfo.<String> get("userId"))));
		}
	}, pageable);
	List<SimpleMemberDto> items = new ArrayList<SimpleMemberDto>();
	List<AcctPo> list = page.getContent();
	for (AcctPo acct : list) {
		SimpleMemberDto dto = new SimpleMemberDto();
		UserPo userPo = acct.getUserPo();
		StringBuilder label = new StringBuilder();
		String sname = userPo.getSname();
		String mobile = userPo.getMobile();
		mobile = null == mobile ? "":mobile;
		String acctNo = acct.getAcctNo();
		dto.setAcctNo(acctNo);
		dto.setSname(sname);
		dto.setMobile(mobile);
		label.append("姓名:");
		label.append(sname);
		label.append(",手机号:");
		label.append(mobile);
		label.append(",交易账号:");
		label.append(acctNo);
		dto.setLabel(label.toString());
		items.add(dto);
	}
	return items;
}


/**
 * 
 */
public List<GroupUserInfoDto> getInvsUsers(final GroupUserSearchDto searchDto) {
	Specification<UserPo> spec = new Specification<UserPo>() {
		@Override
		public Predicate toPredicate(Root<UserPo> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
			Predicate predicate = cb.conjunction();
			List<Expression<Boolean>> expressions = predicate.getExpressions();
			expressions.add(cb.not(cb.exists(getSubQuery(query, cb, searchDto.getGroupId(), root.<String> get("userId")))));
			if(searchDto!=null){
				if(StringUtils.isNotBlank(searchDto.getKeyWord())){
					expressions.add(cb.or(
						cb.like(root.<String>get("sname"), "%"+searchDto.getKeyWord()+"%"),
						cb.like(root.<String>get("nickName"), "%"+searchDto.getKeyWord()+"%"),
						cb.like(root.<String>get("mobile"), "%"+searchDto.getKeyWord()+"%"),
						cb.exists(getWorkUnitSubQuery(query, cb, searchDto.getKeyWord(), root.<String> get("userId")))
					));
				}
				else{
					// 未传关键字查询条件，则查询结果为空
					expressions.add(cb.equal(root.<String>get("userId"),"0"));
				}
			}
			return predicate;
		}
		private Subquery<String> getSubQuery(CriteriaQuery<?> query, 
				CriteriaBuilder cb, String groupId, Path<String> userId) {
			Subquery<String> subQuery = query.subquery(String.class);
			Root<GroupUserPo> groupUsers = subQuery.from(GroupUserPo.class);
			return subQuery.select(groupUsers.<String>get("userId")).where(
					cb.and(cb.equal(groupUsers.<String> get("groupId"), groupId)), 
					cb.equal(groupUsers.<String> get("userId"), userId));
		}
		private Subquery<String> getWorkUnitSubQuery(CriteriaQuery<?> query, 
				CriteriaBuilder cb, String keyWord, Path<String> userId) {
			Subquery<String> subQuery = query.subquery(String.class);
			Root<PersonalInfoPo> personalInfo = subQuery.from(PersonalInfoPo.class);
			return subQuery.select(personalInfo.<String>get("userId")).where(
					cb.equal(personalInfo.<String> get("userId"), userId),
					cb.like(personalInfo.<String> get("workUnit"), "%"+searchDto.getKeyWord()+"%"));
		}
		
	};
	List<UserPo> userList = userRepository.findAll(spec);
	List<GroupUserInfoDto> dtoList = new ArrayList<GroupUserInfoDto>();
	for(UserPo ur:userList){
		GroupUserInfoDto dto = new GroupUserInfoDto();
		dto.setName(ur.getSname());
		dto.setNickName(ur.getNickName());
		dto.setUserType(ur.getUserType());
		dto.setUserId(ur.getUserId());
		if(ur.getUserType() == EUserType.PERSON){
			PersonalInfoPo persnoalInfo = personalInfoRepository.findOne(ur.getUserId());
			if(persnoalInfo!=null){
				dto.setWorkUnit(persnoalInfo.getWorkUnit());
			}
		}
		else{
			dto.setWorkUnit(null);
		}
		dtoList.add(dto);
	}
	return dtoList;
}
	

/**
 * json
 */
private Specification<Qfjbxxdz> getWhereClause(final JSONArray condetion,final JSONArray search) {
	return new Specification<Qfjbxxdz>() {
		@Override
		public Predicate toPredicate(Root<Qfjbxxdz> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
			List<Predicate> predicate = new ArrayList<>();
			Iterator<JSONObject> iterator = condetion.iterator();
			Predicate preP = null;
			while(iterator.hasNext()){
				JSONObject jsonObject = iterator.next();
				//注意：这里用的root.join 因为我们要用qfjbxx对象里的字段作为条件就必须这样做join方法有很多重载，使用的时候可以多根据自己业务决断
				Predicate p1 = cb.equal(root.join("qfjbxx").get("id").as(String.class),jsonObject.get("fzId").toString());
				Predicate p2 = cb.equal(root.get("fzcc").as(String.class),jsonObject.get("ccId").toString());
				if(preP!=null){
					preP = cb.or(preP,cb.and(p1,p2));
				}else{
					preP = cb.and(p1,p2);
				}
			}
			JSONObject jsonSearch=(JSONObject) search.get(0);
			Predicate p3=null;
			if(null!=jsonSearch.get("xm")&&jsonSearch.get("xm").toString().length()>0){
			   p3=cb.like(root.join("criminalInfo").get("xm").as(String.class),"%"+jsonSearch.get("xm").toString()+"%");
			}
			Predicate p4=null;
			if(null!=jsonSearch.get("fzmc")&&jsonSearch.get("fzmc").toString().length()>0){
				p4=cb.like(root.join("qfjbxx").get("fzmc").as(String.class),"%"+jsonSearch.get("fzmc").toString()+"%");
			}
			Predicate preA;
			if(null!=p3&&null!=p4){
				Predicate  preS =cb.and(p3,p4);
				preA =cb.and(preP,preS);
			}else if(null==p3&&null!=p4){
				preA=cb.and(preP,p4);
			}else if(null!=p3&&null==p4){
				preA=cb.and(preP,p3);
			}else{
				preA=preP;
			}
			predicate.add(preA);
			Predicate[] pre = new Predicate[predicate.size()];
			query.where(predicate.toArray(pre));
			return query.getRestriction();
		}
	};
}


/**
 * 
 */
private List<SubAcctTrxJnlPo> getRechargeList(final String trxDate, final String serviceCenterId) {
	return subAcctTrxJnlRepository.findAll(new Specification<SubAcctTrxJnlPo>() {
		@Override
		public Predicate toPredicate(Root<SubAcctTrxJnlPo> root, CriteriaQuery<?> query,
				CriteriaBuilder cb) {
			Predicate predicate = cb.conjunction();
			List<Expression<Boolean>> expressions = predicate.getExpressions();
			expressions
					.add(cb.exists(getSubQuery(query, cb, root.<String> get("acctNo"), serviceCenterId)));
			expressions.add(cb.equal(root.<String> get("trxDate"), trxDate));
			expressions.add(cb.equal(root.<String> get("useType"), EUseType.RECHARGE));
			return predicate;
		}

		private Subquery<String> getSubQuery(CriteriaQuery<?> query, CriteriaBuilder cb,
				Path<String> acctNo, String serviceCenterId) {
			Subquery<String> subQuery = query.subquery(String.class);
			Root<AcctPo> rAcct = subQuery.from(AcctPo.class);
			Root<InvestorInfoPo> rInvestor = subQuery.from(InvestorInfoPo.class);
			return subQuery.select(rAcct.<String> get("acctNo"))
					.where(cb.and(cb.equal(rAcct.<String> get("acctNo"), acctNo),
							cb.equal(rAcct.<String> get("userId"), rInvestor.<String> get("userId")),
							cb.equal(rInvestor.<String> get("serviceCenterId"), serviceCenterId)));
		}
	});
}


/**
 * 
 */
@Transactional(readOnly = true)
public List<AcctDto> getAcctInfoList(final List<String> userIdList, 
		final List<String> groupIds, final List<EBizRole> bizRoles){
	Specification<AcctPo> spec = new Specification<AcctPo>() {
		@Override
		public Predicate toPredicate(Root<AcctPo> root, CriteriaQuery<?> query,	CriteriaBuilder cb) {
			Predicate predicate = cb.conjunction();
			List<Expression<Boolean>> expressions = predicate.getExpressions();
			if(userIdList!=null&&!userIdList.isEmpty()){
				expressions.add(root.<String>get("userId").in(userIdList));
			}
			if(bizRoles!=null && !bizRoles.isEmpty()){
				expressions.add(cb.exists(getRolesSubQuery(query, cb, 
						root.<String>get("userId"), getRoleIds(bizRoles))));
			}
			if(groupIds!=null && !groupIds.isEmpty()){
				expressions.add(cb.exists(getGroupsSubQuery(query, cb, 
						root.<String>get("userId"), getGroupIds(groupIds))));
			}
			return predicate;
		}
		private Subquery<String> getRolesSubQuery(CriteriaQuery<?> query, 
				CriteriaBuilder cb, Path<String> userId, Set<String> roleIds) {
			Subquery<String> subQuery = query.subquery(String.class);
			Root<UserRolePo> param = subQuery.from(UserRolePo.class);
			return subQuery.select(param.<String>get("userId")).where(
					cb.and(cb.equal(param.get("userId"), userId),
						   param.<String>get("roleId").in(roleIds)
			));
		}
		private Subquery<String> getGroupsSubQuery(CriteriaQuery<?> query, 
				CriteriaBuilder cb, Path<String> userId, Set<String> groupIds) {
			Subquery<String> subQuery = query.subquery(String.class);
			Root<GroupUserPo> param = subQuery.from(GroupUserPo.class);
			return subQuery.select(param.<String>get("userId")).where(
					cb.and(cb.equal(param.get("userId"), userId),
						   param.<String>get("groupId").in(groupIds)
			));
		}
		private Set<String> getRoleIds(List<EBizRole> bizRoles){
			Set<String> sets = new HashSet<String>();
			for(EBizRole role:bizRoles){
				sets.add(role.getRoleId());
			}
			return sets;
		}
		private Set<String> getGroupIds(List<String> groupIds){
			Set<String> sets = new HashSet<String>();
			for(String gid:groupIds){
				sets.add(gid);
			}
			return sets;
		}
	};
	List<AcctPo> list = acctRepo.findAll(spec);
	return packItems(list);
}
```